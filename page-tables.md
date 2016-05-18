# Page Tables

__Certainty Level:__ Moderate, very early in development, but this is pretty
fundamental stuff.

__Architecture:__ These notes assume an x86-64 UMA system unless otherwise
specified. [Linux 4.6][linux-4.6] is always targeted.

## Introduction

* [Virtual memory][virtual-memory], used by the majority of modern operating
  systems, uses the [MMU][mmu] hardware (for modern systems, read: CPU) of the
  system to allow [ring][ring]-0 privileged code to specify mappings between
  memory addresses and actual physical memory addresses via a CPU control
  register, rendering the actual addresses used by running code 'virtual'.

* The mappings are performed at the granularity of [page][page]s,
  i.e. contiguous blocks of memory of a set size. In x86-64 you can have page
  sizes of 4KiB, 2MiB and 1GiB, though typically 4KiB (2^12) will be used in
  most cases.

* The kernel switches out virtual memory mappings between userland processes to
  provide an abstraction layer and to isolate processes for each other, however
  this means that each process must store its own set of mappings. If the entire
  address space were mapped an x86-64 system would require 256GiB per process
  consisting overwhelming of null mappings.

* To avoid such egregious space requirements, mappings are subdivided into a
  'sparse' arrangement of [page table][page-table]s and the virtual address
  becomes a set of offsets into these tables along with an offset into the final
  physical page of memory (more explanation on what I mean by 'sparse' below.)

## Page Table Layout

* In linux there are 4 'levels' of page tables. A table consists of an array of
  entries of type `pXX_t`, wrapping a `pXXval_t`:

1. Page Global Directory (PGD) - [pgd_t][pgd_t]/[pgdval_t][pgdval_t].

2. Page Upper Directory (PUD) - [pud_t][pud_t]/[pudval_t][pudval_t].

3. Page Middle Directory (PMD) - [pmd_t][pmd_t]/[pmdval_t][pmdval_t].

4. Page Table Entry directory (PTE) - [pte_t][pte_t]/[pteval_t][pteval_t].

* These types are simply `typedef`'d wrappers around fundamental
  architecture-dependent types, gathering all the x86-64 types together:

```c
typedef unsigned long   pgdval_t;
typedef unsigned long   pudval_t;
typedef unsigned long   pmdval_t;
typedef unsigned long   pteval_t;

typedef struct { pgdval_t pgd; } pgd_t;
typedef struct { pudval_t pud; } pud_t;
typedef struct { pmdval_t pmd; } pmd_t;
typedef struct { pteval_t pte; } pte_t;
```

__NOTE:__ `p[gum]d_t` are defined in
[arch/x86/include/asm/pgtable_types.h][pgtable_types.h], `pte_t` and all the
`pXXval_t` types are defined in
[arch/x86/include/asm/pgtable_64_types.h][pgtable_64_types.h] - this shares as
much as possible between 32 and 64-bit x86.

* As mentioned above, a virtual address is simply a set of offsets into each of
  these tables. In the typical case of a 4KiB page size, each of the PGD, PUD,
  PMD, and PTE tables contain 512 pointers each, and since the word size is 8
  bytes, this means 4KiB of storage i.e. conveniently - each page table takes up
  a page of memory.

* The number of pointers available per table is defined in the `PTRS_PER_Pxx`
  preprocessor constant, i.e. [PTRS_PER_PGD][PTRS_PER_PGD],
  [PTRS_PER_PUD][PTRS_PER_PUD], [PTRS_PER_PMD][PTRS_PER_PMD], and
  [PTRS_PER_PTE][PTRS_PER_PTE].

* The number of bits a virtual address needs to be shifted right before being
  masked to obtain an index into a page table is defined by
  [PGDIR_SHIFT][PGDIR_SHIFT] for PGD, [PUD_SHIFT][PUD_SHIFT] for PUD,
  [PMD_SHIFT][PMD_SHIFT] for PMD and [PAGE_SHIFT][PAGE_SHIFT] for PTE (since at
  that point you just want to 'shift out' the physical page offset.)

* I'll go into the helper functions around this later, but once you've shifted
  an address, you can then simply mask it with `PTRS_PER_Pxx - 1` - this takes
  advantage of the fact that all values less than a power of 2 will be masked by
  the power minus 1 (e.g. `0b1000 - 1 = 0b0111`), so the `_SHIFT` and
  `PTRS_PER_` values are all you need to determine these indexes.

* Gathering all these values together for the typical 4KiB page size case:

```c
#define PGDIR_SHIFT     39
#define PMD_SHIFT       21
#define PUD_SHIFT       30

#define PTRS_PER_PGD    512
#define PTRS_PER_PUD    512
#define PTRS_PER_PMD    512
#define PTRS_PER_PTE    512
```

* And so we can now visualise what an given example address actually represents:

```
    6         5         4         3         2         1
4321098765432109876543210987654321098765432109876543210987654321
0000000000000000000000000000000000000000101100010111000000010000
[   RESERVED   ][  PGD  ][  PUD  ][  PMD  ][  PTE  ][  OFFSET  ]
                         |        |        |        |-PAGE_SHIFT
                         |        |        |-----------PMD_SHIFT
                         |        |--------------------PUD_SHIFT
                         |---------------------------PGDIR_SHIFT
```

* But how does this relate to a 'sparse' layout of pages, what do these offsets
  actually reference?

* In linux, each process is assigned a PGD via
  [struct mm_struct][mm_struct]`->pgd` - if a process (somehow?!) references no
  memory at all it then only requires a single page of memory for mappings,
  rather than the 256GiB of mostly empty space a full set of mappings would
  need.

* Mappings tables are generated as needed and the hierarchy of levels provides a
  'sparse' set of mappings - a page of memory is used for
  [page table][page-table] so, on average (obv. when you run out of space in a
  PTE directory a new one will need to be created and the same goes for PMD and
  PUD also, but these are on average not so common cases), mappings grow by a
  page per 512 new mappings, or ~8 bytes per mapping amortised.

* The best way of showing how this fits together is diagrammatically, so taking
  the example address from above:

```

0000000000000000000000000000000000000000101100010111000000010000
[   RESERVED   ][  PGD  ][  PUD  ][  PMD  ][  PTE  ][  OFFSET  ]

PGD offset =    000000000 = 0
PUD offset =    000000000 = 0
PMD offset =    000000101 = 5
PTE offset =    100010111 = 279
phy offset = 000000010000 = 16


      PGD
    -------
  0 |    -----\        PUD
  . |-----|   |      -------
  . /     /   \--->0 |    -----\        PMD
  . \     \        . |-----|   |      -------
  . /     /        . /     /   \--->0 /     /
  . \     \        . \     \        . \     \
  . /     /        . /     /        . /     /
  . \     \        . \     \        . |-----|
  . /     /        . /     /        5 |    -----\        PTE
512 -------        . \     \        . |-----|   |      -------
                   . /     /        . /     /   \--->0 /     /
                 512 -------        . \     \        . \     \
                                    . /     /        . /     /
                                  512 -------        . |-----|
                                                   279 |    -----\   Phys Page
                                                     . |-----|   |      ---
                                                     . /     /   \--->0 / /
                                                     . \     \        . \ \
                                                     . /     /        . / /
                                                   512 -------        . |-|
                                                                     16 |o|
                                                                      . |-|
                                                                      . |h|
                                                                      . |-|
                                                                      . |a|
                                                                      . |-|
                                                                      . |i|
                                                                      . |-|
                                                                      . |!|
                                                                      . |-|
                                                                      . / /
                                                                      . \ \
                                                                      . / /
                                                                   4096 ---
```

* Since walking these tables is a relatively expensive operation, a cache is
  maintained - the [Translation Lookaside Buffer (TLB)][tlb] - more on this in a
  later section.

### 64-bit Address Space

* In [current x86-64 implementations][x86-64-address-space] only the lower 48
  bits of an address are used - the remaining higher order bits must all be
  equal to the 47th bit, i.e. allowable addresses are 128TiB (47 bits) in the
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

* Taking a quick look at the memory map:

```
...
0000000000000000 - 00007fffffffffff (=47 bits) user space, different per mm
hole caused by [48:63] sign extension
ffff800000000000 - ffff87ffffffffff (=43 bits) guard hole, reserved for hypervisor
ffff880000000000 - ffffc7ffffffffff (=64 TB) direct mapping of all phys. memory
...
```

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
#define __va(x)                 ((void *)((unsigned long)(x)+PAGE_OFFSET))
```

* [__pa()][__pa] is a little more complicated - It invokes
  [__phys_addr()][__phys_addr], which (assuming we haven't set
  `CONFIG_DEBUG_VIRTUAL`) subsequently invokes
  [__phys_addr_nodebug()][__phys_addr_nodebug]:

```c
#define __pa(x)         __phys_addr((unsigned long)(x))

...

#define __phys_addr(x)          __phys_addr_nodebug(x)

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

### Traversing Page Tables

* Since each of the directory tables are page-aligned, [PAGE_SHIFT][PAGE_SHIFT]
  bits will always be 0 for every page table address. This is exploited to allow
  the placing of flags in the lower bits of each entry.

* A consequence of this is that the physical pages read from page tables have to
  be masked to avoid these flags being interpreted as offsets. This can be
  achieved via `&`'ing with [PAGE_MASK][PAGE_MASK], which is simply defined as
  `~(`[PAGE_SIZE][PAGE_SIZE]`-1)` - subtracting 1 from a power of 2 results in a
  mask for that number of possible values, in this case `1000000000000` becomes
  `111111111111`, the `~` complement translates that to
  `1111111111111111111111111111111111111111111111111111000000000000` - masking
  out any flags.

* Each process has an associated [struct mm_struct][mm_struct]:

```c
struct mm_struct {
        ...

        pgd_t * pgd;

        ...
};
```

__NOTE:__ The struct is pretty huge, so limiting to what we care about here -
the PGD for the process :)

* A number of functions are provided to make it easier to traverse page
  tables. It's instructive to have a look at a utility function that performs
  this task, [follow_page()][follow_page] (and subsequently
  [follow_page_mask()][follow_page_mask] and
  [follow_page_pte()][follow_page_pte]) is useful for this task.

* Since the actual `follow_page()` code is large and contains handling for
  various things we're not interested in at this point, I've extracted and
  combined the actual functions to obtain a smaller pedagogical version for the
  simple case (which may contain errors or miss case handling, you've been
  warned), `follow_page_ljs()`:

```c
struct page *follow_page_ljs(struct vm_area_struct *vma,
                             unsigned long address, unsigned int flags)
{
        pgd_t *pgd;
        pud_t *pud;
        pmd_t *pmd;
        spinlock_t *ptl;
        pte_t *ptep, pte;
        struct page *page;
        struct mm_struct *mm = vma->vm_mm;

        pgd = pgd_offset(mm, address);
        if (pgd_none(*pgd) || unlikely(pgd_bad(*pgd)))
                return no_page_table(vma, flags);

        pud = pud_offset(pgd, address);
        if (pud_none(*pud))
                return no_page_table(vma, flags);
        if (unlikely(pud_bad(*pud)))
                return no_page_table(vma, flags);

        pmd = pmd_offset(pud, address);
        if (pmd_none(*pmd))
                return no_page_table(vma, flags);
        if (unlikely(pmd_bad(*pmd)))
                return no_page_table(vma, flags);

        ptep = pte_offset_map_lock(mm, pmd, address, &ptl);
        pte = *ptep;
        if (!pte_present(pte))
                goto no_page;

        page = pfn_to_page(pte_pfn(pte));

        pte_unmap_unlock(ptep, ptl);
        return page;

no_page:
        pte_unmap_unlock(ptep, ptl);
        return no_page_table(vma, flags);
}
```

[linux-4.6]:https://github.com/torvalds/linux/tree/v4.6/

[virtual-memory]:https://en.wikipedia.org/wiki/Virtual_memory
[mmu]:https://en.wikipedia.org/wiki/Memory_management_unit
[ring]:https://en.wikipedia.org/wiki/Protection_ring
[page]:https://en.wikipedia.org/wiki/Page_(computer_memory)
[page-table]:https://en.wikipedia.org/wiki/Page_table
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pgdval_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pte_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[pgtable_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h
[pgtable_64_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[PGDIR_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L27
[PUD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L33
[PMD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L40
[PAGE_SHIFT]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L8
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer

[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
[PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L35
[__PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L35
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__phys_addr_nodebug]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L12
[__START_KERNEL_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L37
[phys_base]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L520
[phys_base-fixup]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L140
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[kdump-paper]:https://www.kernel.org/doc/ols/2007/ols2007v1-pages-167-178.pdf

[PAGE_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L10
[PAGE_SIZE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L9
[follow_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L2186
[follow_page_mask]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L214
[follow_page_pte]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L63
