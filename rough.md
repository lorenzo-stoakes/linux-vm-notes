# Rough Notes

__NOTE:__ These are rough notes intentionally not linked from elsewhere, and may
be full of errors, swearing and Trump quotes.

## General

* `pfn_to_page()`, `pte_pfn()` seem to be useful. `pfn_to_page()` ultimately
  calls `__pfn_to_page()` which varies depending on memory model - COVER.

## 2.4.22 -> 4.6

* Seems like missing page levels is handled via
  [pgtable-nopmd.h][pgtable-nopmd.h] now.

* `CONFIG_PGTABLE_LEVELS` is used to determine number of pg table levels now.

## Big Blocks of Code

* No more `pte_offset()` for x86, can use
  [pte_offset_map_lock()][pte_offset_map_lock] (which assigns and locks a
  spinlock during the operation) which calls [pte_offset_map()][pte_offset_map]
  -> [pte_offset_kernel()][pte_offset_kernel] ->
  [pmd_page_vaddr()][pmd_page_vaddr] -> [pmd_pfn_mask()][pmd_pfn_mask] ->
  [PHYSICAL_PMD_PAGE_MASK][PHYSICAL_PMD_PAGE_MASK] ->
  [PTE_PFN_MASK][PTE_PFN_MASK] -> [PHYSICAL_PAGE_MASK][PHYSICAL_PAGE_MASK] ->
  [PAGE_MASK][PAGE_MASK] -> [__PHYSICAL_MASK][__PHYSICAL_MASK] ->
  [__PHYSICAL_MASK_SHIFT][__PHYSICAL_MASK_SHIFT].

(Note that `_PAGE_PSE` refers to [Page Size Extension (PSE)][pse], i.e. huge
pages again and we are ignoring that for now here :D)

```c
static inline pte_t *pte_offset_kernel(pmd_t *pmd, unsigned long address)
{
        return (pte_t *)pmd_page_vaddr(*pmd) + pte_index(address);
}

static inline unsigned long pmd_page_vaddr(pmd_t pmd)
{
        return (unsigned long)__va(pmd_val(pmd) & pmd_pfn_mask(pmd));
}

static inline pmdval_t pmd_pfn_mask(pmd_t pmd)
{
        if (native_pmd_val(pmd) & _PAGE_PSE)
                return PHYSICAL_PMD_PAGE_MASK;
        else
                return PTE_PFN_MASK;
}

/* Extracts the PFN from a (pte|pmd|pud|pgd)val_t of a 4KB page */
#define PTE_PFN_MASK            ((pteval_t)PHYSICAL_PAGE_MASK)

/* Cast *PAGE_MASK to a signed type so that it is sign-extended if
   virtual addresses are 32-bits but physical addresses are larger
   (ie, 32-bit PAE). */
#define PHYSICAL_PAGE_MASK      (((signed long)PAGE_MASK) & __PHYSICAL_MASK)
#define PHYSICAL_PMD_PAGE_MASK  (((signed long)PMD_PAGE_MASK) & __PHYSICAL_MASK)

#define __PHYSICAL_MASK         ((phys_addr_t)((1ULL << __PHYSICAL_MASK_SHIFT) - 1))

/* See Documentation/x86/x86_64/mm.txt for a description of the memory map. */
#define __PHYSICAL_MASK_SHIFT   46
```

* We can simplify this for the case of non-huge pages, which is equivalent to
  the 2.4.22 version:

```c
static inline pte_t *pte_offset_kernel_ljs(pmd_t *pmd, unsigned long address)
{
        return (pte_t *)(__va((pmd_val(pmd) & (PAGE_MASK & __PHYSICAL_MASK))) +
                pte_index(address));
}
```


* There are new mechanisms in the VM - obv. huge page tables (+ transparent),
  but also 'devmap' and page splitting as referenced in
  [follow_page_mask()][follow_page_mask]:

```c
/**
 * follow_page_mask - look up a page descriptor from a user-virtual address
 * @vma: vm_area_struct mapping @address
 * @address: virtual address to look up
 * @flags: flags modifying lookup behaviour
 * @page_mask: on output, *page_mask is set according to the size of the page
 *
 * @flags can have FOLL_ flags set, defined in <linux/mm.h>
 *
 * Returns the mapped (struct page *), %NULL if no mapping exists, or
 * an error pointer if there is a mapping to something not represented
 * by a page descriptor (see also vm_normal_page()).
 */
struct page *follow_page_mask(struct vm_area_struct *vma,
                              unsigned long address, unsigned int flags,
                              unsigned int *page_mask)
{
        pgd_t *pgd;
        pud_t *pud;
        pmd_t *pmd;
        spinlock_t *ptl;
        struct page *page;
        struct mm_struct *mm = vma->vm_mm;

        *page_mask = 0;

        page = follow_huge_addr(mm, address, flags & FOLL_WRITE);
        if (!IS_ERR(page)) {
                BUG_ON(flags & FOLL_GET);
                return page;
        }

        pgd = pgd_offset(mm, address);
        if (pgd_none(*pgd) || unlikely(pgd_bad(*pgd)))
                return no_page_table(vma, flags);

        pud = pud_offset(pgd, address);
        if (pud_none(*pud))
                return no_page_table(vma, flags);
        if (pud_huge(*pud) && vma->vm_flags & VM_HUGETLB) {
                page = follow_huge_pud(mm, address, pud, flags);
                if (page)
                        return page;
                return no_page_table(vma, flags);
        }
        if (unlikely(pud_bad(*pud)))
                return no_page_table(vma, flags);

        pmd = pmd_offset(pud, address);
        if (pmd_none(*pmd))
                return no_page_table(vma, flags);
        if (pmd_huge(*pmd) && vma->vm_flags & VM_HUGETLB) {
                page = follow_huge_pmd(mm, address, pmd, flags);
                if (page)
                        return page;
                return no_page_table(vma, flags);
        }
        if ((flags & FOLL_NUMA) && pmd_protnone(*pmd))
                return no_page_table(vma, flags);
        if (pmd_devmap(*pmd)) {
                ptl = pmd_lock(mm, pmd);
                page = follow_devmap_pmd(vma, address, pmd, flags);
                spin_unlock(ptl);
                if (page)
                        return page;
        }
        if (likely(!pmd_trans_huge(*pmd)))
                return follow_page_pte(vma, address, pmd, flags);

        ptl = pmd_lock(mm, pmd);
        if (unlikely(!pmd_trans_huge(*pmd))) {
                spin_unlock(ptl);
                return follow_page_pte(vma, address, pmd, flags);
        }
        if (flags & FOLL_SPLIT) {
                int ret;
                page = pmd_page(*pmd);
                if (is_huge_zero_page(page)) {
                        spin_unlock(ptl);
                        ret = 0;
                        split_huge_pmd(vma, pmd, address);
                } else {
                        get_page(page);
                        spin_unlock(ptl);
                        lock_page(page);
                        ret = split_huge_page(page);
                        unlock_page(page);
                        put_page(page);
                }

                return ret ? ERR_PTR(ret) :
                        follow_page_pte(vma, address, pmd, flags);
        }

        page = follow_trans_huge_pmd(vma, address, pmd, flags);
        spin_unlock(ptl);
        *page_mask = HPAGE_PMD_NR - 1;
        return page;
}
```

* Might be worth exploring new fields in [struct mm_struct][mm_struct]:

```c
struct mm_struct {
        struct vm_area_struct *mmap;            /* list of VMAs */
        struct rb_root mm_rb;
        u32 vmacache_seqnum;                   /* per-thread vmacache */
#ifdef CONFIG_MMU
        unsigned long (*get_unmapped_area) (struct file *filp,
                                unsigned long addr, unsigned long len,
                                unsigned long pgoff, unsigned long flags);
#endif
        unsigned long mmap_base;                /* base of mmap area */
        unsigned long mmap_legacy_base;         /* base of mmap area in bottom-up allocations */
        unsigned long task_size;                /* size of task vm space */
        unsigned long highest_vm_end;           /* highest vma end address */
        pgd_t * pgd;
        atomic_t mm_users;                      /* How many users with user space? */
        atomic_t mm_count;                      /* How many references to "struct mm_struct" (users count as 1) */
        atomic_long_t nr_ptes;                  /* PTE page table pages */
#if CONFIG_PGTABLE_LEVELS > 2
        atomic_long_t nr_pmds;                  /* PMD page table pages */
#endif
        int map_count;                          /* number of VMAs */

        spinlock_t page_table_lock;             /* Protects page tables and some counters */
        struct rw_semaphore mmap_sem;

        struct list_head mmlist;                /* List of maybe swapped mm's.  These are globally strung
                                                 * together off init_mm.mmlist, and are protected
                                                 * by mmlist_lock
                                                 */


        unsigned long hiwater_rss;      /* High-watermark of RSS usage */
        unsigned long hiwater_vm;       /* High-water virtual memory usage */

        unsigned long total_vm;         /* Total pages mapped */
        unsigned long locked_vm;        /* Pages that have PG_mlocked set */
        unsigned long pinned_vm;        /* Refcount permanently increased */
        unsigned long data_vm;          /* VM_WRITE & ~VM_SHARED & ~VM_STACK */
        unsigned long exec_vm;          /* VM_EXEC & ~VM_WRITE & ~VM_STACK */
        unsigned long stack_vm;         /* VM_STACK */
        unsigned long def_flags;
        unsigned long start_code, end_code, start_data, end_data;
        unsigned long start_brk, brk, start_stack;
        unsigned long arg_start, arg_end, env_start, env_end;

        unsigned long saved_auxv[AT_VECTOR_SIZE]; /* for /proc/PID/auxv */

        /*
         * Special counters, in some configurations protected by the
         * page_table_lock, in other configurations by being atomic.
         */
        struct mm_rss_stat rss_stat;

        struct linux_binfmt *binfmt;

        cpumask_var_t cpu_vm_mask_var;

        /* Architecture-specific MM context */
        mm_context_t context;

        unsigned long flags; /* Must use atomic bitops to access the bits */

        struct core_state *core_state; /* coredumping support */
#ifdef CONFIG_AIO
        spinlock_t                      ioctx_lock;
        struct kioctx_table __rcu       *ioctx_table;
#endif
#ifdef CONFIG_MEMCG
        /*
         * "owner" points to a task that is regarded as the canonical
         * user/owner of this mm. All of the following must be true in
         * order for it to be changed:
         *
         * current == mm->owner
         * current->mm != mm
         * new_owner->mm == mm
         * new_owner->alloc_lock is held
         */
        struct task_struct __rcu *owner;
#endif

        /* store ref to file /proc/<pid>/exe symlink points to */
        struct file __rcu *exe_file;
#ifdef CONFIG_MMU_NOTIFIER
        struct mmu_notifier_mm *mmu_notifier_mm;
#endif
#if defined(CONFIG_TRANSPARENT_HUGEPAGE) && !USE_SPLIT_PMD_PTLOCKS
        pgtable_t pmd_huge_pte; /* protected by page_table_lock */
#endif
#ifdef CONFIG_CPUMASK_OFFSTACK
        struct cpumask cpumask_allocation;
#endif
#ifdef CONFIG_NUMA_BALANCING
        /*
         * numa_next_scan is the next time that the PTEs will be marked
         * pte_numa. NUMA hinting faults will gather statistics and migrate
         * pages to new nodes if necessary.
         */
        unsigned long numa_next_scan;

        /* Restart point for scanning and setting pte_numa */
        unsigned long numa_scan_offset;

        /* numa_scan_seq prevents two threads setting pte_numa */
        int numa_scan_seq;
#endif
#if defined(CONFIG_NUMA_BALANCING) || defined(CONFIG_COMPACTION)
        /*
         * An operation with batched TLB flushing is going on. Anything that
         * can move process memory needs to flush the TLB when moving a
         * PROT_NONE or PROT_NUMA mapped page.
         */
        bool tlb_flush_pending;
#endif
        struct uprobes_state uprobes_state;
#ifdef CONFIG_X86_INTEL_MPX
        /* address of the bounds directory */
        void __user *bd_addr;
#endif
#ifdef CONFIG_HUGETLB_PAGE
        atomic_long_t hugetlb_usage;
#endif
```

[pfn_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L81

[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[follow_page_mask]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L214

[follow_page_pte]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L63
[pte_offset_map_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1680
[pte_offset_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L140
[pte_offset_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L600
[pmd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L557
[pmd_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L329
[PHYSICAL_PMD_PAGE_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L25
[PTE_PFN_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L242
[PHYSICAL_PAGE_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L24
[PAGE_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L10
[__PHYSICAL_MASK]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L18
[__PHYSICAL_MASK_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L40
[pse]:https://en.wikipedia.org/wiki/Page_Size_Extension
