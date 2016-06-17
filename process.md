# Process Address Space

## 64-bit Address Space

* Let's take a look at the _virtual_ [x86-64 memory map][x86-64-mm]:

```
Userland (128TiB)

                        0000000000000000 -> |---------------| ^
                                            |    Process    | |
                                            |    address    | | 128 TiB
                                            |     space     | |
                        0000800000000000 -> |---------------| v

             .        ` .     -                 `-       ./   _
                      _    .`   -   The netherworld of  `/   `
            -     `  _        |  /      unavailable sign-extended -/ .
             ` -        .   `  48-bit address space  -     \  /    -
           \-                - . . . .             \      /       -

Kernel (128TiB)

                        ffff800000000000 -> |----------------| ^
                                            |   Hypervisor   | |
                                            |    reserved    | | 8 TiB
                                            |      space     | |
        __PAGE_OFFSET = ffff880000000000 -> |----------------| x
                                            | Direct mapping | |
                                            |  of all phys.  | | 64 TiB
                                            |     memory     | |
                        ffffc80000000000 -> |----------------| v
                                            /                /
                                            \      hole      \
                                            /                /
        VMALLOC_START = ffffc90000000000 -> |----------------| ^
                                            |    vmalloc/    | |
                                            |    ioremap     | | 32 TiB
                                            |     space      | |
      VMALLOC_END + 1 = ffffe90000000000 -> |----------------| v
                                            /                /
                                            \      hole      \
                                            /                /
        VMEMMAP_START = ffffea0000000000 -> |----------------| ^
                                            |     Virtual    | |
                                            |   memory map   | | 1 TiB
                                            |  (struct page  | |
                                            |     array)     | |
                        ffffeb0000000000 -> |----------------| v
                                            /                /
                                            \    'unused'    \
                                            /      hole      /
                                            \                \
                        ffffec0000000000 -> |----------------| ^
                                            |  Kasan shadow  | | 16 TiB
                                            |     memory     | |
                        fffffc0000000000 -> |----------------| v
                                            /                /
                                            \    'unused'    \
                                            /      hole      /
                                            \                \
     ESPFIX_BASE_ADDR = ffffff0000000000 -> |----------------| ^
                                            |   %esp fixup   | | 512 GiB
                                            |     stacks     | |
                        ffffff8000000000 -> |----------------| v
                                            /                /
                                            \    'unused'    \
                                            /      hole      /
                                            \                \
           EFI_VA_END = ffffffef00000000 -> |----------------| ^
                                            |   EFI region   | | 64 GiB
                                            | mapping space  | |
         EFI_VA_START = ffffffff00000000 -> |----------------| v
                                            /                /
                                            \    'unused'    \
                                            /      hole      /
                                            \                \
   __START_KERNEL_map = ffffffff80000000 -> |----------------| ^
                                            |  Kernel text   | | 512 MiB
                                            |    mapping     | |
        MODULES_VADDR = ffffffffa0000000 -> |----------------| x
                                            |     Module     | |
                                            |    mapping     | | 1.5 GiB
                                            |     space      | |
                        ffffffffff600000 -> |----------------| x
                                            |   vsyscalls    | | 8 MiB
                        ffffffffffe00000 -> |----------------| v
                                            /                /
                                            \    'unused'    \
                                            /      hole      /
                                            \                \
                                            ------------------
```

* In [current x86-64 implementations][x86-64-address-space] only the lower 48
  bits of an address are used - the remaining higher order bits must all be
  equal to the 48th bit, i.e. allowable addresses are 128TiB (47 bits) in the
  `0000000000000000` - `00007fffffffffff` range, and 128TiB (47 bits) in the
  `ffff800000000000` - `ffffffffffffffff` range - the address space is divided
  into upper and lower portions.

* In fact, reading the [x86-64 memory map doc][x86-64-mm], only 46 bits (64TiB)
  of RAM is supported - this makes sense, as it allows for the entire physical
  address space to be mapped into the kernel while leaving another 64TiB free
  for other kernel data.

* In linux, like most (if not all?) other operating systems, this provides a
  nice separation between kernel and user address space for free - keep kernel
  addresses in the upper portion and user addresses in the lower portion.

### Kernel Address Translation

* We can see there's a gap reserved for the hypervisor between ffff800000000000
  and ffff87ffffffffff, immediately after which the entire 64TiB physical
  address space is mapped.

* This allows us to have a simple means of translating from a physical address
  to a virtual one within the kernel - simply offset the physical one by
  `ffff880000000000`. This constant is defined as [PAGE_OFFSET][PAGE_OFFSET] (an
  alias for [__PAGE_OFFSET][__PAGE_OFFSET].)

* More generally to translate from virtual to physical addresses and vice-versa
  in the kernel, two functions are available - [phys_to_virt()][phys_to_virt] (a
  wrapper around [__va()][__va]) and [virt_to_phys()][virt_to_phys] (a wrapper
  around [__pa()][__pa].)

* Give that we have `PAGE_OFFSET` [__va()][__va] is simple:

```c
#define __va(x) ((void *)((unsigned long)(x)+PAGE_OFFSET))
```

* [__pa()][__pa] is a little more complicated - It invokes
  [__phys_addr()][__phys_addr], which (assuming we haven't set
  `CONFIG_DEBUG_VIRTUAL`) subsequently invokes
  [__phys_addr_nodebug()][__phys_addr_nodebug]:

```c
#define __pa(x) __phys_addr((unsigned long)(x))

...

#define __phys_addr(x) __phys_addr_nodebug(x)

...

static inline unsigned long __phys_addr_nodebug(unsigned long x)
{
        unsigned long y = x - __START_KERNEL_map;

        /* use the carry flag to determine if x was < __START_KERNEL_map */
        x = y + ((x > y) ? phys_base : (__START_KERNEL_map - PAGE_OFFSET));

        return x;
}
```

* What is [__START_KERNEL_map][__START_KERNEL_map] (which is defined as
  `ffffffff80000000`)? We can see from the [memory map][x86-64-mm] that this is
  the virtual address above which the kernel is loaded:

```
...
ffffffff80000000 - ffffffffa0000000 (=512 MB)  kernel text mapping, from phys 0
ffffffffa0000000 - ffffffffff5fffff (=1526 MB) module mapping space
ffffffffff600000 - ffffffffffdfffff (=8 MB) vsyscalls
ffffffffffe00000 - ffffffffffffffff (=2 MB) unused hole
```

* [__phys_addr_nodebug()][__phys_addr_nodebug] therefore differentiates between
  virtual addresses that have been mapped from the direct physical mapping at
  [__PAGE_OFFSET][__PAGE_OFFSET] and those that originate from the kernel itself
  from [__START_KERNEL_map][__START_KERNEL_map] on.

* One important thing to note here is that [phys_base][phys_base] is used to
  offset the returned address if it is indeed a reference to a kernel symbol -
  this is a value that is [determined on startup][phys_base-fixup] in case the
  kernel is loaded higher than expected. This might occur if the kernel is
  loaded via [kdump][kdump] for example (more details are available in
  [Kdump: Smarter, Easier, Trustier (PDF)][kdump-paper], a paper on the
  subject.)

[PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L35
[__PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L35
[__START_KERNEL_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L37
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__phys_addr_nodebug]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L12
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[kdump-paper]:https://www.kernel.org/doc/ols/2007/ols2007v1-pages-167-178.pdf
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[phys_base-fixup]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L140
[phys_base]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L520
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
