# Page Tables

## Introduction

* [Virtual memory][virtual-memory], used by the majority of modern operating
  systems, uses the [MMU][mmu] hardware (for modern systems, read: CPU) of the
  system to allow [ring][ring]-0 privileged code to specify mappings between
  memory addresses and actual physical memory addresses via a CPU control
  register, rendering the actual addresses used by running code 'virtual'.

* The mappings are performed at the granularity of [page][page/ref]s,
  i.e. contiguous blocks of memory of a set size. In x86-64 you can have page
  sizes of 4KiB, 2MiB and 1GiB, though typically 4KiB (2^12) will be used in
  most cases.

* The kernel switches out virtual memory mappings between userland processes to
  provide an abstraction layer and to isolate processes for each other, however
  this means that each process must store its own set of mappings. If the entire
  address space were mapped an x86-64 system would require 256GiB per process
  consisting overwhelmingly of null mappings.

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

* Once you've shifted an address, you can then simply mask it with
  `PTRS_PER_Pxx - 1` - this takes advantage of the fact that all values less
  than a power of 2 will be masked by the power minus 1 (e.g. `0b1000 - 1 =
  0b0111`), so the `_SHIFT` and `PTRS_PER_` values are all you need to determine
  these indexes. Typically this kind of operation is performed using helper
  [functions][funcs] however.

* Gathering all these values together for the typical 4KiB page size case:

```c
#define PGDIR_SHIFT     39
#define PUD_SHIFT       30
#define PMD_SHIFT       21
#define PAGE_SHIFT      12

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

* Taking another quick look at the [memory map][x86-64-mm]:

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

## Page Table Entry Flags

* Since each of the directory tables are page-aligned, [PAGE_SHIFT][PAGE_SHIFT]
  bits will always be 0 for every page table address. This is exploited to allow
  the placing of flags in the lower bits of each entry, and in x86-64, where
  there are only 46 addressable bits, flags are also placed in the higher bits.

* A consequence of this is that the physical pages read from page tables have to
  be masked to avoid these flags being interpreted as offsets. In the case of
  4KiB pages, this is achieved via [PTE_PFN_MASK][PTE_PFN_MASK] and
  [PTE_FLAGS_MASK][PTE_FLAGS_MASK]. Larger page sizes needed some special
  handling, however we won't go into that here.

* [PTE_FLAGS_MASK][PTE_FLAGS_MASK] is simply the bitwise complement of
  [PTE_PFN_MASK][PTE_PFN_MASK], so we need only consider how the latter
  functions to understand both:

```
PAGE_MASK = ~((1UL << 12)-1) =
1111111111111111111111111111111111111111111111111111000000000000

__PHYSICAL_MASK = ((1UL << 46)-1) =
0000000000000000001111111111111111111111111111111111111111111111

PTE_PFN_MASK = PAGE_MASK & __PHYSICAL_MASK =
1111111111111111111111111111111111111111111111111111000000000000 &
0000000000000000001111111111111111111111111111111111111111111111 =
0000000000000000001111111111111111111111111111111111000000000000

PTE_FLAGS_MASK = ~PTE_PFN_MASK =
1111111111111111110000000000000000000000000000000000111111111111
```

* As discussed above, we can see here that clearly we are masking out/in the
  lower [PAGE_SHIFT][PAGE_SHIFT] (12) bits, as well as the unaddressable higher
  bits.

* These extra bits are used to store a number of different flags indicating the
  state of the memory page whose physical address is contained in the page table
  entry. This point is subtle and important to avoiding confusion - a
  [pgd_t][pgd_t] entry's flags refer to the PUD page that PGD entry is pointing
  at, so each `pXX_t` entry value actually describes a `pXX` a level lower.

* The [functions][funcs] page contains a list of helper functions for
  interacting with these flags. Generally they don't need to be manually
  manipulated, rather the `pXX_<flag>()` and `pXX_mk<flag>()` functions can be
  used.

* Examining the more commonly used page flags (all taken from
  [arch/x86/include/asm/pgtable_types.h][pgtable_types.h]):

1. `_PAGE_PRESENT` - Determines whether the page is available in memory rather
   than swapped out or otherwise unavailable.
2. `_PAGE_RW` - If cleared, the memory page is read-only.
3. `_PAGE_USER` - If cleared, the memory can only be accessed by the kernel.
4. `_PAGE_ACCESSED` - The page has been accessed - this is a 'sticky bit', and
   if left cleared when a page is created, the first access to the page will set
   the flag and it will remain set until manually cleared.
5. `_PAGE_DIRTY` - The page has been modified - this is a 'sticky bit', and if
   left cleared when a page is created, the first write to the page will set the
   flag and it will remain set until manually cleared.
6. `_PAGE_PSE` - Indicates the page is a huge page, i.e. either 1GiB or 2MiB
   rather than 4KiB.
7. `_PAGE_GLOBAL` - Prevents ordinary [TLB][tlb] flushes from evicting this
   page's mapping from the TLB.

## Process PGDs and Context Switches

* Each process has an associated [struct mm_struct][mm_struct] which describes
  its general memory state including a pointer to its PGD page, which gives us
  the means to traverse page tables starting with this.

* The PGD pointer is interesting - it's stored as a `pgd_t *pgd` field in the
  [struct mm_struct][mm_struct], and rather than being a pointer to a separate
  PGD entry somewhere, this field actually contains the _virtual_ address of the
  process's PGD page, i.e. it's being treated like a pointer to the start of an
  array. Its physical address can be obtained using [__pa()][__pa] if being set
  in the CPU, or if the kernel needs to traverse the page tables manually the
  virtual address can be used directly.

* On x86-64, each time the kernel context switches to a process it does this by
  writing the process's PGD, i.e. `*current->mm->pgd` to the `cr3`
  register. Doing this flushes the [TLB][tlb] by default as of course one
  process's cached mappings aren't useful for another's, unless those pages are
  marked `_PAGE_GLOBAL` in which case the mappings are not flushed.

* On each context switch the kernel's own static mappings are preserved as each
  process has its PGD configured with these included - this way switching
  between kernel and user mode is a lot less costly - no need for TLB flushes
  and PGD switching. The mappings are protected from unauthorised userland usage
  by simply clearing the `_PAGE_USER` flag. These mappings are marked with
  `_PAGE_GLOBAL` so TLB flushes do not invalidate them.

## Traversing Page Tables

* A number of functions are provided to make it easier to traverse page
  tables. It's instructive to have a look at a utility function that performs
  this task, [__follow_pte()][__follow_pte] is useful for this task:

```c
static int __follow_pte(struct mm_struct *mm, unsigned long address,
                pte_t **ptepp, spinlock_t **ptlp)
{
        pgd_t *pgd;
        pud_t *pud;
        pmd_t *pmd;
        pte_t *ptep;

        pgd = pgd_offset(mm, address);
        if (pgd_none(*pgd) || unlikely(pgd_bad(*pgd)))
                goto out;

        pud = pud_offset(pgd, address);
        if (pud_none(*pud) || unlikely(pud_bad(*pud)))
                goto out;

        pmd = pmd_offset(pud, address);
        VM_BUG_ON(pmd_trans_huge(*pmd));
        if (pmd_none(*pmd) || unlikely(pmd_bad(*pmd)))
                goto out;

        /* We cannot handle huge page PFN maps. Luckily they don't exist. */
        if (pmd_huge(*pmd))
                goto out;

        ptep = pte_offset_map_lock(mm, pmd, address, ptlp);
        if (!ptep)
                goto out;
        if (!pte_present(*ptep))
                goto unlock;
        *ptepp = ptep;
        return 0;
unlock:
        pte_unmap_unlock(ptep, *ptlp);
out:
        return -EINVAL;
}
```

* The [functions][funcs] section contains detailed descriptions of each of the
  functions used here. Note that the `_offset()` functions are confusingly
  named - they actually provide the virtual address of the required page table
  entry which contains the physical address of either the page table or the
  final physical page the entry refers to.

* Note that we have to check in each case for the entry being empty (`_none()`
  functions), unsuitable for use (`_bad()` functions) and the final PTE being
  swapped out or otherwise unavailable (`pte_present()`.)

## Translating Between Page Table Entries and Physical Page Descriptors

* Each physical page of memory in the system is described by a
  [struct page][page]. See the [Physical Pages][physical] section for more
  details on this.

* A number of functions are available for translating between addresses, page
  table entries and [struct page][page]s, listed over in the [functions][funcs]
  section. A key one of these is [virt_to_page()][virt_to_page] which translates
  a virtual kernel address to its corresponding [struct page][page].

* Additionally, page table pages can be retrieved using the `pXX_page()` family
  of functions - for example, [pgd_page()][pgd_page] retrieves the
  [struct page][page] describing the PUD entry pointed at by the specified PGD
  entry.

* Note that as with many page table functions, there is a confusing relation
  between its name and what is returned - each `pXX_page()` functions returns a
  reference to the _underlying_ page the specified entry refers to, so
  [pgd_page()][pgd_page] returns a [struct page][page] describing a PUD page,
  [pud_page()][pud_page] returns a [struct page][page] describing a PMD page,
  [pmd_page()][pmd_page] returns a [struct page][page] describing a PTE page,
  and finally [pte_page()][pte_page] returns a [struct page][page] describing
  the ultimate physical page the overall page table hierarchy refers to.

* [struct page][page]s can be translated back to a physical address via
  [page_to_phys()][page_to_phys], or via [page_to_pfn()][page_to_pfn], which
  returns a Page Frame Number (PFN.) If you imagine a contiguous array of
  [struct page][page]s, the PFN would be the index of a given page in that
  array. We can therefore simply translate from a physical address to a PFN by
  shifting right by [PAGE_SHIFT][PAGE_SHIFT], or vice-versa by shifting left.

## Translation Lookaside Buffer (TLB)

* The traversal of the page table hierarchy described here is performed
  automatically by the CPU each time a virtual address is referenced (and since,
  when paging is enabled, _every_ address is virtual these really translates as
  'an address is referenced') but only after the
  [Translation Lookaside Buffer (TLB)][tlb] is consulted to see if the virtual
  to physical address mapping isn't already cached there.

* So, in essence, the TLB is simply a cache designed to avoid as much as
  possible the expensive operation of walking the page tables to translate a
  virtual address to a physical one.

* As mentioned previously, every time a [context switch][context-switch] occurs,
  which on x86 means the `cr3` register is written to to point at a different
  PGD, the TLB is flushed except for pages marked `_PAGE_GLOBAL` (shared kernel
  mappings are marked this way.) This makes sense, as the majority of different
  processes' non-kernel mappings will be entirely different and maintaining a
  TLB cache for this makes no sense.

* The TLB can also be flushed manually via a number of [functions][funcs], for
  example [flush_tlb_mm()][flush_tlb_mm] will flush the TLB entries for the
  specified [struct mm_struct][mm_struct].

* The finer grained `invlpg` instruction on 486+ x86 CPUs allows for the
  invalidation of individual page mappings, which makes sense when a partial
  flush is required, however if a full flush is needed, this can simply be
  achieved by reading then writing the `cr3` register.

* The trade-off between `invlpg` and a full flush is discussed in the
  [Documentation/x86/tlb.txt][doc-tlb] file.

* Where paravirtualisation is in effect, e.g. when running [xen][xen], TLB
  flushing performs a hypervisor call rather than actually attempting to flush
  the TLB natively.

* Since each individual CPU can run a different process, the TLB is maintained
  per-CPU. The manual TLB flush functions take this into account by flushing all
  CPUs that use the specified [struct mm_struct][mm_struct], and in the case of
  full flushes these are performed on all CPUs via
  [Inter-Processor Interrupts (IPI)][ipi].

* The cost of a TLB miss is between 10-100 clock cycles, with a miss rate of
  around 0.1-1% (ref: [wikipedia article][tlb].)

### Lazy TLB

* Kernel threads do not have their own [struct mm_struct][mm_struct] descriptor
  as they do not have their own set of address mappings (i.e. their
  [struct task_struct][task_struct]`->mm` field is `NULL`.)

* To efficiently gain access to kernel mappings when a kernel thread is switched
  to by the scheduler, the previous process's [struct mm_struct][mm_struct] is
  assigned to the `active_mm` field, and the PGD is _not_ swapped (from
  [kernel/sched/core.c][context_switch-lazytlb]:)

```c
static __always_inline struct rq *
context_switch(struct rq *rq, struct task_struct *prev,
               struct task_struct *next)
{
        ...

        if (!mm) {
                next->active_mm = oldmm;
                atomic_inc(&oldmm->mm_count);
                enter_lazy_tlb(oldmm, next);
        } else
                switch_mm(oldmm, mm, next);

        ...
}
```

* Consequently, no TLB flush occurs. This is called 'lazy TLB' as we avoid an
  expensive TLB flush operation on this context switch. We need to have access
  to the [struct mm_struct][mm_struct] we're using, regardless of the kernel
  thread rightly not having its `->mm` task field set, so we use `active_mm` to
  keep a track of this.

* Any attempt at a manual TLB flush results in [swapper_pg_dir][swapper_pg_dir],
  the static kernel mappings only PGD initialised at boot, being set as the
  process's PGD via [leave_mm()][leave_mm]. At this point the 'lazy' TLB is
  forced into action :)

[PAGE_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L10
[PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L35
[PAGE_SHIFT]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L8
[PAGE_SIZE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L9
[PGDIR_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L27
[PMD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L40
[PTE_FLAGS_MASK]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L248
[PTE_PFN_MASK]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L242
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[PUD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L33
[__PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L35
[__START_KERNEL_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L37
[__follow_pte]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3594
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__phys_addr_nodebug]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L12
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[context-switch]:https://en.wikipedia.org/wiki/Context_switch
[context_switch-lazytlb]:https://github.com/torvalds/linux/blob/v4.6/kernel/sched/core.c#L2731
[doc-tlb]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/tlb.txt
[flush_tlb_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L293
[ipi]:https://en.wikipedia.org/wiki/Inter-processor_interrupt
[kdump-paper]:https://www.kernel.org/doc/ols/2007/ols2007v1-pages-167-178.pdf
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[leave_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L41
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mmu]:https://en.wikipedia.org/wiki/Memory_management_unit
[page-table]:https://en.wikipedia.org/wiki/Page_table
[page/ref]:https://en.wikipedia.org/wiki/Page_(computer_memory)
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[page_to_pfn]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L80
[page_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L144
[pgd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L676
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pgdval_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pgtable_64_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h
[pgtable_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h
[phys_base-fixup]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L140
[phys_base]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L520
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[pmd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L566
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pte_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L171
[pte_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[pud_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L635
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[ring]:https://en.wikipedia.org/wiki/Protection_ring
[swapper_pg_dir]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L25
[task_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L1394
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[virt_to_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L63
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[virtual-memory]:https://en.wikipedia.org/wiki/Virtual_memory
[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
[xen]:https://en.wikipedia.org/wiki/Xen

[funcs]:./page-table-funcs.md
[physical]:./physical.md
