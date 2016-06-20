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

## Memory Descriptors

* Each process's memory state is represented by a 'memory descriptor',
  [struct mm_struct][mm_struct], which specifies the process's PGD (see
  [page tables][page-tables]), amongst a number of other fields.

* Looking at the [struct mm_struct][mm_struct], assuming x86-64 with a pretty
  standard configuration in order to strip  irrelevant `CONFIG_xxx` fields:

```c
struct mm_struct {
        struct vm_area_struct *mmap;            /* list of VMAs */
        struct rb_root mm_rb;
        u32 vmacache_seqnum;                   /* per-thread vmacache */

        unsigned long (*get_unmapped_area) (struct file *filp,
                                unsigned long addr, unsigned long len,
                                unsigned long pgoff, unsigned long flags);

        unsigned long mmap_base;                /* base of mmap area */
        unsigned long mmap_legacy_base;         /* base of mmap area in bottom-up allocations */
        unsigned long task_size;                /* size of task vm space */
        unsigned long highest_vm_end;           /* highest vma end address */
        pgd_t * pgd;
        atomic_t mm_users;                      /* How many users with user space? */
        atomic_t mm_count;                      /* How many references to "struct mm_struct" (users count as 1) */
        atomic_long_t nr_ptes;                  /* PTE page table pages */
        atomic_long_t nr_pmds;                  /* PMD page table pages */

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

        spinlock_t                      ioctx_lock;
        struct kioctx_table __rcu       *ioctx_table;

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

        /* store ref to file /proc/<pid>/exe symlink points to */
        struct file __rcu *exe_file;

        struct mmu_notifier_mm *mmu_notifier_mm;

        /*
         * An operation with batched TLB flushing is going on. Anything that
         * can move process memory needs to flush the TLB when moving a
         * PROT_NONE or PROT_NUMA mapped page.
         */
        bool tlb_flush_pending;

        struct uprobes_state uprobes_state;

        atomic_long_t hugetlb_usage;
};
```

## Virtual Memory Areas

* Each process's virtual memory is additionally divided into non-overlapping
  regions (Virtual Memory Areas or 'VMA's) related by their purpose and
  protection state.

* VMAs are represented by the [struct vm_area_struct][vm_area_struct] type.

* A [struct mm_struct][mm_struct]'s VMAs are stored both as a doubly-linked list
  and a [red-black tree][red-black], the head of the linked list being kept in
  the `mm_struct`'s `mmap` field, and previous/next nodes kept in the
  [struct vm_area_struct][vm_area_struct]'s `vm_prev` and `vm_next` fields,
  sorted in address order, and the red/black root in the `mm_struct`'s `mm_rb`
  field, with the node kept in the `vm_area_struct`'s `vm_rb` field.

* Looking at the [struct vm_area_struct][vm_area_struct] (again assuming x86-64
  with a pretty standard configuration in order to strip irrelevant `CONFIG_xxx`
  fields):

```c
struct vm_area_struct {
        /* The first cache line has the info for VMA tree walking. */

        unsigned long vm_start;         /* Our start address within vm_mm. */
        unsigned long vm_end;           /* The first byte after our end address
                                           within vm_mm. */

        /* linked list of VM areas per task, sorted by address */
        struct vm_area_struct *vm_next, *vm_prev;

        struct rb_node vm_rb;

        /*
         * Largest free memory gap in bytes to the left of this VMA.
         * Either between this VMA and vma->vm_prev, or between one of the
         * VMAs below us in the VMA rbtree and its ->vm_prev. This helps
         * get_unmapped_area find a free area of the right size.
         */
        unsigned long rb_subtree_gap;

        /* Second cache line starts here. */

        struct mm_struct *vm_mm;        /* The address space we belong to. */
        pgprot_t vm_page_prot;          /* Access permissions of this VMA. */
        unsigned long vm_flags;         /* Flags, see mm.h. */

        /*
         * For areas with an address space and backing store,
         * linkage into the address_space->i_mmap interval tree.
         */
        struct {
                struct rb_node rb;
                unsigned long rb_subtree_last;
        } shared;

        /*
         * A file's MAP_PRIVATE vma can be in both i_mmap tree and anon_vma
         * list, after a COW of one of the file pages.  A MAP_SHARED vma
         * can only be in the i_mmap tree.  An anonymous MAP_PRIVATE, stack
         * or brk vma (with NULL file) can only be in an anon_vma list.
         */
        struct list_head anon_vma_chain; /* Serialized by mmap_sem &
                                          * page_table_lock */
        struct anon_vma *anon_vma;      /* Serialized by page_table_lock */

        /* Function pointers to deal with this struct. */
        const struct vm_operations_struct *vm_ops;

        /* Information about our backing store: */
        unsigned long vm_pgoff;         /* Offset (within vm_file) in PAGE_SIZE
                                           units */
        struct file * vm_file;          /* File we map to (can be NULL). */
        void * vm_private_data;         /* was vm_pte (shared mem) */

        struct mempolicy *vm_policy;    /* NUMA policy for the VMA */

        struct vm_userfaultfd_ctx vm_userfaultfd_ctx;
};
```

[PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L35
[__PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L35
[__START_KERNEL_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L37
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__phys_addr_nodebug]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L12
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[kdump-paper]:https://www.kernel.org/doc/ols/2007/ols2007v1-pages-167-178.pdf
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[phys_base-fixup]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L140
[phys_base]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L520
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[red-black]:https://en.wikipedia.org/wiki/Red%E2%80%93black_tree
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[vm_area_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L294
[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt

[page-tables]:./page-tables.md
