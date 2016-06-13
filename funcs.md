# VM Functions

## Contents

### Address Translation

* [phys_to_virt()](#phys_to_virt) - Translates a physical address to a virtual
  one.
* [virt_to_phys()](#virt_to_phys) - Translates a virtual address to a physical
  one.
* [virt_to_page()](#virt_to_page) - Retrieves the [struct page][page] that
  describes the page containing the specified virtual address.
* [page_to_pfn()](#page_to_pfn) - Converts a [struct page][page] to its
  corresponding Page Frame Number (PFN.)
* [page_to_phys()](#page_to_phys) - Retrieve the physical address of the page
  described by the specified [struct page][page].
* [pfn_to_page()](#pfn_to_page) - Converts a Page Frame Number (PFN) to its
  corresponding [struct page][page].
* [__va()](#__va) - Translates a physical address to a virtual one.
* [__pa()](#__pa) - Translates a virtual address to a physical one.

### Utility Functions

* [virt_addr_valid()](#virt_addr_valid) - Determines if a specified virtual
  address is a valid _kernel_ virtual address.
* [pfn_valid()](#pfn_valid) - Determines if a specified Page Frame Number (PFN)
  represents a valid physical address.
* [pte_same()](#pte_same) - Determines whether the two specified PTE entries
  refer to the same physical page and flags.

### TLB

* [flush_tlb()](#flush_tlb) - Flushes the current
  [struct mm_struct][mm_struct]'s [TLB][tlb] entries.
* [flush_tlb_all()](#flush_tlb_all) - Flushes all processes [TLB][tlb] entries.
* [flush_tlb_mm()](#flush_tlb_mm) - Flushes the specified
  [struct mm_struct][mm_struct]'s [TLB][tlb] entries.
* [flush_tlb_page()](#flush_tlb_page) - Flushes a single
  [struct vm_area_struct][vm_area_struct]-specified page's [TLB][tlb] entry.
* [flush_tlb_range()](#flush_tlb_range) - Flushes a range of
  [struct vm_area_struct][vm_area_struct]-specified addresses [TLB][tlb]
  entries.
* [flush_tlb_mm_range()](#flush_tlb_mm_range) - Flushes a range of
  [struct mm_struct][mm_struct]'s [TLB][tlb] entries.
* [flush_tlb_kernel_range()](#flush_tlb_kernel_range) - Flushes a range of
  kernel pages.
* [flush_tlb_others()](#flush_tlb_others) - Flushes a range of
  [struct mm_struct][mm_struct]'s [TLB][tlb] entries on other CPUs.

---

### phys_to_virt()

`void *phys_to_virt(phys_addr_t address)`

[phys_to_virt()][phys_to_virt] translates physical address `address` to a
kernel-mapped virtual one.

Wrapper around [__va()][__va].

#### Arguments

* `address` - Physical address to be translated.

#### Returns

Kernel-mapped virtual address.

---

### virt_to_phys()

`phys_addr_t virt_to_phys(volatile void *address)`

[virt_to_phys()][virt_to_phys] translate kernel-mapped virtual address `address`
to a physical one.

Wrapper around [__pa()][__pa].

#### Arguments

* `address` - Kernel-mapped virtual address to be translated.

#### Returns

Physical address associated with specified virtual address.

---

### __va()

`void *__va(phys_addr_t address)`

[__va()][__va] does the heavy lifting for `phys_to_virt()`, converting a
physical memory address to a kernel-valid virtual one.

This function simply adds the kernel memory offset `PAGE_OFFSET`,
`0xffff880000000000` for x86-64. Looking at the [x86-64 memory map][x86-64-mm],
you can see this simply provides a virtual address that is part of the kernel's
_complete_ direct mapping of all physical memory between `0xffff880000000000`
and `0xffffc7ffffffffff`.

The function performs no checks and assumes the supplied physical memory address
is valid and in the 64TiB of allowed memory in current x86-64 linux kernel
implementations.

__NOTE:__ Macro, inferring function signature.

---

### __pa()

`phys_addr_t __pa(volatile void *address)`

[__pa()][__pa] does the heavy lifting for `virt_to_phys()`, converting a
_kernel_ virtual memory address to a physical one. The function is a wrapper
around [__phys_addr()][__phys_addr].

The function isn't quite as simple as [__va()][__va] as it has to determine
whether the supplied virtual address is part of the kernel's direct mapping of
_all_ of the physical memory between `0xffff880000000000` and
`0xffffc7ffffffffff`, or whether it's part of the kernel's 'text' from
`__START_KERNEL_map` on (`0xffffffff80000000` for [x86-64][x86-64-mm]), and
offsets the supplied virtual address accordingly.

In the case of the address originating from the kernel's 'text', the physical
offset of the kernel, `phys_base` is taken into account, which allows for the
kernel to be loaded in a different physical location e.g. when
[kdump][kdump]ing.

__NOTE:__ Macro, inferring function signature.

---

### virt_to_page()

`struct page *virt_to_page(unsigned long kaddr)`

[virt_to_page()][virt_to_page] determines the physical address of the specified
kernel virtual address, then the Page Frame Number (PFN) of the physical page
that contains it, and finally passes this to [pfn_to_page()][pfn_to_page] to
retrieve the [struct page][page] which describes the physical page.

__Important:__ As per the code comment above the function, the returned pointer
is valid if and only if [virt_addr_valid(kaddr)][virt_addr_valid] returns
`true`.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `kaddr` - The virtual _kernel_ address whose [struct page][page] we desire.

#### Returns

The [struct page][page] describing the physical page the specified virtual
address resides in.

---

### page_to_pfn()

`unsigned long page_to_pfn(struct page *page)`

[page_to_pfn()][page_to_pfn] returns the Page Frame Number (PFN) that is
associated with the specified [struct page][page].

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

How the PFN is determined varies depending on the memory model, in x86-64 UMA
this is [__page_to_pfn()][__page_to_pfn] under `CONFIG_SPARSEMEM_VMEMMAP` - the
memory map is virtually contiguous at `vmemmap`, (`0xffffea0000000000`, see
[x86-64 memory map][x86-64-mm].)

This makes the implementation of the function straightforward - simply subtract
`vmemmap` from the page pointer (being careful with typing to have pointer
arithmetic take into account `sizeof(struct page)` for you.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `page` - The [struct page][page] whose corresponding Page Frame Number (PFN)
  is desired.

#### Returns

The PFN of the specified [struct page][page].

---

### page_to_phys()

`dma_addr_t page_to_phys(struct page *page)`

[page_to_phys()][page_to_phys] returns a physical address for the page described
by the specified [struct page][page].

Oddly it seems to return a [dma_addr_t][dma_addr_t], possibly due to use for
device I/O (it is declared in `arch/x86/include/asm/io.h`), however in x86-64 it
makes no difference as `dma_addr_t` and [phys_addr_t][phys_addr_t] are the
same - unsigned long (64-bit.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `page` - The [struct page][page] whose physical start address we desire.

#### Returns

The physical address of the start of the page which the specified
[struct page][page] describes.

---

### pfn_to_page()

`struct page *pfn_to_page(unsigned long pfn)`

[pfn_to_page()][pfn_to_page] returns the [struct page][page] that is associated
with the specified Page Frame Number (PFN.)

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

How the [struct page][page] is located varies depending on the memory model, in
x86-64 UMA this is [__pfn_to_page()][__pfn_to_page] under
`CONFIG_SPARSEMEM_VMEMMAP` - the memory map is virtually contiguous at
`vmemmap`, (`0xffffea0000000000`, see [x86-64 memory map][x86-64-mm].)

This makes the implementation of the function straightforward - simply offset
the PFN by `vmemmap` (being careful with typing to have pointer arithmetic take
into account `sizeof(struct page)` for you.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pfn` - The Page Frame Number (PFN) whose corresponding [struct page][page] is
  desired.

#### Returns

The [struct page][page] that describes the physical page with specified PFN.

### virt_addr_valid()

`bool virt_addr_valid(unsigned long kaddr)`

[virt_addr_valid()][virt_addr_valid] determines if the specified virtual address
`kaddr` is actually a valid, non-[vmalloc][vmalloc]'d kernel address.

The function is a wrapper for [__virt_addr_valid()][__virt_addr_valid], which,
once its checked the virtual address is in a valid range, checks it has a valid
corresponding physical PFN via [pfn_valid()][pfn_valid].

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `kaddr` - Virtual address which we want to determine is a valid non-vmalloc'd
  kernel address or not.

#### Returns

`true` if the address is valid, `false` if not.

---

### pfn_valid()

`int pfn_valid(unsigned long pfn)`

[pfn_valid()][pfn_valid] determine whether the specified Page Frame Number (PFN)
is valid, i.e. in x86-64 whether it refers to a valid 46-bit address, and
whether there is actually physical memory mapped to that physical location.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pfn` - PFN whose validity we wish to determine.

#### Returns

Truthy (non-zero) if the PFN is valid, 0 if not.

---

### pte_same()

`int pte_same(pte_t a, pte_t b)`

[pte_same()][pte_same] determines whether the two specified PTE entries refer to
the same physical page AND share the same flags.

On x86-64 it's as simple as `a.pte == b.pte`.

#### Arguments

* `a` - The first PTE entry whose physical page address and flags we want to
  compare.

* `b` - The second PTE entry whose physical page address and flags we want to
  compare.

#### Returns

1 if the PTE entries' physical address and flags are the same, 0 if not.

---

### flush_tlb()

`void flush_tlb(void)`

[flush_tlb()][flush_tlb] is a wrapper around
[flush_tlb_current_task()][flush_tlb_current_task] which flushes the current
[struct mm_struct][mm_struct] [TLB][tlb] mappings.

[flush_tlb_current_task()][flush_tlb_current_task] checks whether any other CPUs
use the current [struct mm_struct][mm_struct], and if so invokes
[flush_tlb_others()][flush_tlb_others] to flush the TLB entries for those CPUs
too.

Ultimately the flushing is performed by [local_flush_tlb()][local_flush_tlb]
which is a wrapper around [__flush_tlb()][__flush_tlb] which itself wraps
[__native_flush_tlb()][__native_flush_tlb] which flushes the CPU's TLB by simply
reading and writing back the contents of the `cr3` register.

__NOTE:__ Macro, inferring function signature.

#### Arguments

N/A

#### Returns

N/A

---

### flush_tlb_all()

`void flush_tlb_all(void)`

[flush_tlb_all()][flush_tlb_all] flushes the [TLB][tlb] entries for all
processes on all CPUs.

Note that this function causes mappings that have `_PAGE_GLOBAL` to be evicted
also, using the `invpcid` instruction on modern CPUs (via
[__flush_tlb_all()][__flush_tlb_all], [__flush_tlb_global()][__flush_tlb_global]
and subsequently [invpcid_flush_all()][invpcid_flush_all].)

#### Arguments

N/A

#### Returns

N/A

---

### flush_tlb_mm()

`void flush_tlb_mm(struct mm_struct *mm)`

[flush_tlb_mm()][flush_tlb_mm] is simply a wrapper around
[flush_tlb_mm_range()][flush_tlb_mm_range] specifying that the whole memory
address space `mm` describes is to be flushed, with no flags specified.

See the description of `flush_tlb_mm_range()` below for a description of the
implementation of the flush.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] whose TLB entries we want flushed.

#### Returns

N/A

---

### flush_tlb_page()

`void flush_tlb_page(struct vm_area_struct *vma, unsigned long start)`

[flush_tlb_page()][flush_tlb_page] flushes a single page's TLB mapping at the
specified address.

If [struct vm_area_struct][vm_area_struct] specified by `vma` does _not_ refer
to the active [struct mm_struct][mm_struct], then the operation is a no-op on
this CPU, as there's nothing to flush.

If the current process is a kernel thread (i.e. `current->mm == NULL`), which
indicates the current CPU is in a lazy TLB mode, we invoke
[leave_mm()][leave_mm] to switch out the [struct mm_struct][mm_struct] we are
'borrowing' and we're done.

Otherwise, ultimately the `invlpg` instruction is invoked (ignoring the
paravirtualised case where a hypervisor function is called directly) via
[__flush_tlb_one()][__flush_tlb_one],
[__flush_tlb_single()][__flush_tlb_single], and
[__native_flush_tlb_single()][__native_flush_tlb_single].

Intel's documentation on the `invlpg` instruction indicates that the specified
address does not need to be page aligned, and in the case of pages larger than
4KiB in size with multiple TLBs for that page, all will be safely
flushed. Additionally, under certain circumstances more or even all TLB entries
may be flushed, however this is presumably unlikely.

#### Arguments

* `vma` - The [struct vm_area_struct][vm_area_struct] which contains the
  [struct mm_struct][mm_struct] which in turn describes the page whose TLB entry
  we want to flush.

* `start` - A virtual address contained in the page we want to flush.

#### Returns

N/A

---

### flush_tlb_range()

```c
void flush_tlb_range(struct vm_area_struct *vma, unsigned long start,
                     unsigned long end)
```

[flush_tlb_range()][flush_tlb_range] is a wrapper around
[flush_tlb_mm_range()][flush_tlb_mm_range], it uses the
[struct mm_struct][mm_struct] belonging to the specified
[struct vm_area_struct][vm_area_struct] `vma`, i.e. `vma->vm_mm`, as well as the
vma's flags, i.e. `vma->vm_flags` and simply passes these on:

```c
#define flush_tlb_range(vma, start, end)        \
                flush_tlb_mm_range(vma->vm_mm, start, end, vma->vm_flags)
```

See [flush_tlb_mm_range()][flush_tlb_mm_range] below for more details as to how
the range flush is achieved.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `vma` - The [struct vm_area_struct][vm_area_struct] which contains the range
  of addresses we wish to [TLB][tlb] flush.

* `start` - The start of the range of virtual addresses we want to [TLB][tlb]
  flush.

* `end` - The exclusive upper bound of the range of virtual addresses we wish to
  [TLB][tlb] flush.

#### Returns

N/A

---

### flush_tlb_mm_range()

```c
void flush_tlb_mm_range(struct mm_struct *mm, unsigned long start,
                        unsigned long end, unsigned long vmflag)
```

[flush_tlb_mm_range()][flush_tlb_mm_range] causes a [TLB][tlb] flush to occur in
the virtual address range specified (note that `end` is an exclusive bound.)

If the current process's [struct task_struct][task_struct]'s `active_mm` field
doesn't match the specified `mm` argument, there's nothing to do so we exit.

If the current process is a kernel thread (i.e. `current->mm == NULL`), which
indicates the current CPU is in a lazy TLB mode, we invoke
[leave_mm()][leave_mm] to switch out the [struct mm_struct][mm_struct] we are
'borrowing' and we're done.

If the `vmflag` flag field has its `VM_HUGETLB` bit set, i.e. the page range
includes huge pages, or the range is specified to be a full flush range
(i.e. `end == TLB_FLUSH_ALL`), then a full flush is performed.

Otherwise, the number of pages to flush is compared to the
[tlb_single_page_flush_ceiling][tlb_single_page_flush_ceiling] variable, which
is set at `33`. If the number of pages to flush exceeds this value, then a full
flush is performed instead. The [Documentation/x86/tlb.txt][tlb.txt]
documentation goes into more detail as to the trade off between a full flush and
individual flushes, and the code comment above this value explains:

```c
/*
 * See Documentation/x86/tlb.txt for details.  We choose 33
 * because it is large enough to cover the vast majority (at
 * least 95%) of allocations, and is small enough that we are
 * confident it will not cause too much overhead.  Each single
 * flush is about 100 ns, so this caps the maximum overhead at
 * _about_ 3,000 ns.
 *
 * This is in units of pages.
 */
```

If any of the conditions for a full flush are met,
[local_flush_tlb()][local_flush_tlb] is called.

Otherwise, each page is flushed via [__flush_tlb_single()][__flush_tlb_single],
and ultimately the x86 `invlpg` instruction is invoked to perform the page-level
flushing, unless paravirtualisation (e.g. [xen][xen]) is in place in which case
a hypervisor function is called directly.

If the [struct mm_struct][mm_struct] is in use by other CPUs,
[flush_tlb_others()][flush_tlb_others] is invoked to perform a TLB flush for
those CPUs also.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] which contains the range of addresses
  we wish to [TLB][tlb] flush.

* `start` - The start of the range of virtual addresses we want to [TLB][tlb]
  flush.

* `end` - The exclusive upper bound of the range of virtual addresses we wish to
  [TLB][tlb] flush.

* `vmflag` - The [struct vm_area_struct][vm_area_struct] flags associated with
  this region of memory, used only to determine if `VM_HUGETLB` is set.

#### Returns

N/A

---

### flush_tlb_kernel_range()

`void flush_tlb_kernel_range(unsigned long start, unsigned long end)`

[flush_tlb_kernel_range()][flush_tlb_kernel_range] [TLB][tlb] flushes the
specified range of kernel virtual addresses.

If the `end` argument is set to `TLB_FLUSH_ALL` or the specified address range
exceeds the [tlb_single_page_flush_ceiling][tlb_single_page_flush_ceiling]
variable (see description of `flush_tlb_mm_range()` above for more details on
this), a global flush is performed on each CPU via
[__flush_tlb_all()][__flush_tlb_all]. This is necessary as kernel mappings are
marked `_PAGE_GLOBAL`.

Otherwise, individual pages are flushed one-by-one via
[__flush_tlb_single()][__flush_tlb_single].

#### Arguments

* `start` - The start of the range of virtual kernel addresses we want to
  [TLB][tlb] flush.

* `end` - The exclusive upper bound of the range of virtual kernel addresses we
  wish to [TLB][tlb] flush.

#### Returns

N/A

---

### flush_tlb_others()

```c
void flush_tlb_others(const struct cpumask *cpumask,
                      struct mm_struct *mm, unsigned long start,
                      unsigned long end)
```

[flush_tlb_others()][flush_tlb_others] flushes the [TLB][tlb] for CPUs other
than the one invoking the call. It's used by the other [TLB][tlb] flush
functions to ensure that TLB flushes are performed across all pertinent CPUs
(i.e. each CPU which reference a [struct mm_struct][mm_struct] or in the case of
full flushes, all CPUs.)

Other CPUs are made to performing a flush via
[Inter-Processor Interrupts (IPIs)][ipi] using
[smp_call_function_many()][smp_call_function_many] each of which invoke
[flush_tlb_func()][flush_tlb_func].

[flush_tlb_func()][flush_tlb_func] operates as follows:

If the `current->active_mm` [struct mm_struct][mm_struct] is not the same as the
one requested to be flushed, the function exits. Additionally, if the CPU TLB
state is set to lazy TLB, [leave_mm()][leave_mm] is invoked to switch out the
[struct mm_struct][mm_struct] we are 'borrowing' and we're done.

Otherwise, it is checked whether a full flush is requested, if so this is
performed via [local_flush_tlb()][local_flush_tlb]. If a full flush is not
requested, individual pages are evicted via
[__flush_tlb_single()][__flush_tlb_single]. Interestingly, no check against
[tlb_single_page_flush_ceiling][tlb_single_page_flush_ceiling] is performed,
presumably because [flush_tlb_others()][flush_tlb_others] seems only to be used
by other manual [TLB][tlb] flushing functions and presumably they would have
already checked for this.

[flush_tlb_func()][flush_tlb_func] has some egregious timing concerns because of
the invocation of an [IPI][ipi] and the possibility of a
[struct mm_struct][mm_struct] being switched out from under the function via
[switch_mm()][switch_mm], as described by its comment:

```c
/*
 * The flush IPI assumes that a thread switch happens in this order:
 * [cpu0: the cpu that switches]
 * 1) switch_mm() either 1a) or 1b)
 * 1a) thread switch to a different mm
 * 1a1) set cpu_tlbstate to TLBSTATE_OK
 *      Now the tlb flush NMI handler flush_tlb_func won't call leave_mm
 *      if cpu0 was in lazy tlb mode.
 * 1a2) update cpu active_mm
 *      Now cpu0 accepts tlb flushes for the new mm.
 * 1a3) cpu_set(cpu, new_mm->cpu_vm_mask);
 *      Now the other cpus will send tlb flush ipis.
 * 1a4) change cr3.
 * 1a5) cpu_clear(cpu, old_mm->cpu_vm_mask);
 *      Stop ipi delivery for the old mm. This is not synchronized with
 *      the other cpus, but flush_tlb_func ignore flush ipis for the wrong
 *      mm, and in the worst case we perform a superfluous tlb flush.
 * 1b) thread switch without mm change
 *      cpu active_mm is correct, cpu0 already handles flush ipis.
 * 1b1) set cpu_tlbstate to TLBSTATE_OK
 * 1b2) test_and_set the cpu bit in cpu_vm_mask.
 *      Atomically set the bit [other cpus will start sending flush ipis],
 *      and test the bit.
 * 1b3) if the bit was 0: leave_mm was called, flush the tlb.
 * 2) switch %%esp, ie current
 *
 * The interrupt must handle 2 special cases:
 * - cr3 is changed before %%esp, ie. it cannot use current->{active_,}mm.
 * - the cpu performs speculative tlb reads, i.e. even if the cpu only
 *   runs in kernel space, the cpu could load tlb entries for user space
 *   pages.
 *
 * The good news is that cpu_tlbstate is local to each cpu, no
 * write/read ordering problems.
 */

/*
 * TLB flush funcation:
 * 1) Flush the tlb entries if the cpu uses the mm that's being flushed.
 * 2) Leave the mm if we are in the lazy tlb mode.
 */
```

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `cpumask` - The mask representing the CPUs which are to be [TLB][tlb] flushed.

* `mm` - The [struct mm_struct][mm_struct] which contains the range of addresses
  we wish to [TLB][tlb] flush.

* `start` - The start of the range of virtual addresses we want to [TLB][tlb]
  flush.

* `end` - The exclusive upper bound of the range of virtual addresses we wish to
  [TLB][tlb] flush.

#### Returns

N/A

---

[__flush_tlb]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L62
[__flush_tlb_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L182
[__flush_tlb_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L63
[__flush_tlb_one]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L190
[__flush_tlb_single]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L64
[__native_flush_tlb]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L136
[__native_flush_tlb_single]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L177
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__page_to_pfn]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L54
[__pfn_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L53
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[__virt_addr_valid]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/physaddr.c#L86
[dma_addr_t]:https://github.com/torvalds/linux/blob/v4.6/include/linux/types.h#L152
[flush_tlb]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L305
[flush_tlb_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L280
[flush_tlb_current_task]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L163
[flush_tlb_func]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L101
[flush_tlb_kernel_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L296
[flush_tlb_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L293
[flush_tlb_mm_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L192
[flush_tlb_others]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L323
[flush_tlb_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L245
[flush_tlb_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L295
[invpcid_flush_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L48
[ipi]:https://en.wikipedia.org/wiki/Inter-processor_interrupt
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[leave_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L41
[local_flush_tlb]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L291
[mm_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[page-table]:https://en.wikipedia.org/wiki/Page_table
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[page_to_pfn]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L80
[page_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L144
[pfn_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L81
[pfn_valid]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1140
[phys_addr_t]:https://github.com/torvalds/linux/blob/v4.6/include/linux/types.h#L162
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[pte_same]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L486
[smp_call_function_many]:https://github.com/torvalds/linux/blob/v4.6/kernel/smp.c#L403
[switch_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/mmu_context.h#L118
[task_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L1394
[tlb.txt]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/tlb.txt
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[tlb_single_page_flush_ceiling]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L190
[virt_addr_valid]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L66
[virt_to_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L63
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[vm_area_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L294
[vmalloc]:http://www.makelinux.net/books/lkd2/ch11lev1sec5
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
[xen]:https://en.wikipedia.org/wiki/Xen
