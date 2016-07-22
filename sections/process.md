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

### Fields

* `struct vm_area_struct *mmap` - The first VMA in a linked-list of all the
  memory descriptor's VMAs (see below diagram for more details.)

* `struct rb_root mm_rb` - Root of the [red-black tree][red-black] containing
  VMAs for fast lookup.

* `u32 vmacache_seqnum` - The [VMA cache][vma-cache] uses sequence numbers
  stored in both the [struct mm_struct][mm_struct] and the
  [struct task_struct][task_struct] to ensure that cached VMAs have not been
  invalidated for the running threads - if the current task's sequence number
  does not match the memory descriptor's, then the cache is considered invalid
  and flushed. Changes to the address space increment the sequence number in the
  `struct mm_struct`, and thus trigger this cache invalidation.

* `unsigned long (*get_unmapped_area)(struct file *filp, unsigned long addr,
   unsigned long len, unsigned long pgoff, unsigned long flags)` - The function
   used to get an unmapped area with the specified parameters within the memory
   controlled by the descriptor. The actual function used is determined by what
   [arch_pick_mmap_layout()][arch_pick_mmap_layout] has assigned to it -
   [arch_get_unmapped_area_topdown()][arch_get_unmapped_area_topdown] is used if
   there is no reason to avoid assigning the top-most available address (as
   decided by [mmap_is_legacy()][mmap_is_legacy]), or if the top-down lookup
   fails, otherwise an unmapped area is determined bottom-up via
   [arch_get_unmapped_area()][arch_get_unmapped_area].

* `unsigned long mmap_legacy_base` - This is the minimum address to assign
  bottom-up mmap allocations from, determined in
  [arch_pick_mmap_layout()][arch_pick_mmap_layout] - which, consists of a random
  [ASLR][aslr] offset via [arch_mmap_rnd()][arch_mmap_rnd] (if randomisation is
  enabled for this descriptor), added to
  [TASK_UNMAPPED_BASE][TASK_UNMAPPED_BASE]. This constant is set at
  (page-aligned) 1/3 of the maximum userland address. All mmap's begin from this
  base.

* `unsigned long mmap_base` - If in legacy mmap mode, then this is the same as
  `mmap_legacy_base`, otherwise it is set to the (page-aligned) maximum userland
  address subtracting an ASLR random factor (as above) and a gap equal to the
  system stack size, adjusted to fit inside the range [MIN_GAP][MIN_GAP] <=
  `gap` <= [MAX_GAP][MAX_GAP]. In non-legacy mode, this base is used as an
  uppermost limit to assign from. The `MIN_GAP` is at least around 128MiB.

* `unsigned long task_size` - Set to the size of user-space address space,
  [TASK_SIZE][TASK_SIZE] (128TiB - 1 page on x86-64.)

* `unsigned long highest_vm_end` - The highest `vm_end` of any VMA in the
  descriptor, i.e. 1 + the maximum address covered by a VMA in the descriptor
  (it's an exclusive bound.)

* `pgd_t *pgd` - The PGD for this process.

* `atomic_t mm_users` - The reference count of processes that are using the
  userspace portion of memory referenced by the descriptor, typically new
  threads (in linux threads are processes that share a memory descriptor), or
  logic that needs to avoid tear down. Decremented by [mmput()][mmput], which,
  when this count reaches 0, invokes [exit_mmap()][exit_mmap] (and subsequently
  [unmap_vmas()][unmap_vmas]) to free all userspace mappings. Finally `mmput()`
  decrements `mm_count` via [mmdrop()][mmdrop]. Incrementing this count is
  simply performed via [atomic_inc()][atomic_inc].

* `atomic_t mm_count` - The ultimate reference count for the descriptor, with
  all of the `mm_users` counting for 1 here. The key reason there's a separate
  count from `mm_users` is that kernel threads 'borrow' the memory descriptor a
  userland process (see lazy TLB in the [page tables][page-tables] section), and
  don't care that userland has been torn down if `mm_users` has reached 0, but
  obviously need to keep the descriptor around for the kernel
  mappings. `mm_count` is decremented via [mmdrop()][mmdrop], and as with
  `mm_users` it is simply incremented via [atomic_inc()][atomic_inc].

* `atomic_long_t nr_ptes, nr_pmds` - A count of PTEs and PMDs associated with
  the descriptor.

* `int map_count` - The number of VMAs in use in the descriptor.

* `spinlock_t page_table_lock` - A general lock for page tables (though
  [split page table locks][split-page-table-lock] mean PTEs and PMDs have
  finer-grained locking.)

* `struct rw_semaphore mmap_sem` - Protects the VMA list.

* `struct list_head mmlist` - Entry for list which is strung off
  [init_mm][init_mm], protected by [mmlist_lock][mmlist_lock].

* `unsigned long hiwater_rss` - The 'high-watermark' of RSS usage, i.e. the
  highest number of pages seen in the [Resident Set Size (RSS)][rss] (RSS is a
  count of actually faulted in pages) of the process. Interestingly, this field
  is not actually set for efficiency except when RSS is about to be _reduced_
  (see the comment in [task_mem()][task_mem]), so if this value is less than the
  current RSS, then the current RSS should be considered the high-watermark
  since it's only increased so far.

* `unsigned long hiwater_vm` - The 'high-watermark' of `total_vm`, again this is
  only updated when total memory is about to be _reduced_ so if this value is
  less than `total_vm`, then `total_vm` should be considered the high-watermark
  (see the comment in [task_mem()][task_mem].)

* `unsigned long total_vm` - The total number of pages mapped by VMAs in the
  memory descriptor (these may not be faulted in yet however).

* `unsigned long locked_vm` - The number of locked pages referenced by the
  memory descriptor. These pages are never swapped out.

* `unsigned long pinned_vm` - The number of 'pinned' pages referenced by the
  memory descriptor. These pages are those which have had their refcount
  incremented such that they are never moved in physical or virtual memory or
  swapped. A 'pinned' page isn't a formal definition, rather a convention of
  raising the refcount permanently (see [this discussion][vm_pinned-thread] for
  possible future options), rather this counter was
  [introduced][pinned_vm-commit] to prevent double-counting.

* `unsigned long data_vm` - Number of pages of _private_ non-executable,
  non-stack memory mapped by the process (may not yet be faulted in.)

* `unsigned long exec_vm` - Number of pages of _private_ executable memory
  mapped by the process (may not yet be faulted in.)

* `unsigned long stack_vm` - Number of pages of _private_ stack memory mapped by
  the process (may not yet be faulted in.)

* `unsigned long def_flags` - A bit field which can contain only the `VM_LOCKED`
  and `VM_LOCKONFAULT` flags. If one or both are set, then all VMA flags will
  default to having these set. The former ensures mappings are not evictable
  (i.e. swapped out), and by necessary pre-faulted in, the latter allows the
  pages to be faulted in as normal, but locked once they are.

* `unsigned long start_code, end_code, start_data, end_data` - The start address
  and exclusive end of the code and data sections of the process, i.e. `start_*`
  is the address of the first byte of these sections, and `end_*` is 1 byte past
  the last byte of these sections.

* `unsigned long start_brk, brk` - The start address and exclusive end, `brk`, of
  the heap.

* `unsigned long start_stack` - The start address of the stack.

* `unsigned long arg_start, arg_end, env_start, env_end` - The start address and
  exclusive end of the process's command-line arguments and environmental
  variables.

* `unsigned long saved_auxv[AT_VECTOR_SIZE]` - A collection of auxiliary values
  passed to a running process known as [auxv][auxv], or a auxiliary vector. This
  allows passing information from the kernel such as the [VDSO][vdso]
  address. The field is populated in [create_elf_tables()][create_elf_tables].

* `struct mm_rss_stat rss_stat` - A set of statistics contained in
  [struct mm_rss_stat][mm_rss_stat] relating to [Resident Set Size (RSS)][rss],
  i.e. memory that has been faulted in. The structure is simply an array of
  `atomic_long_t` counts for each of: `MM_FILEPAGES` - number of resident pages
  mapping files, `MM_ANONPAGES` - number of resident anonymous pages,
  `MM_SWAPENTS` - number of resident swap entries, and `MM_SHMEMPAGES` - number
  of resident shared pages. Note that in the usual case where the
  `SPLIT_RSS_COUNTING` constant is set, these statistics are only updated every
  [TASK_RSS_EVENTS_THRESH][TASK_RSS_EVENTS_THRESH] page faults (hardcoded to
  64.)

* `struct linux_binfmt *binfmt` - [struct linux_binfmt][linux_binfmt] which
  contains the appropriate binary format handler for the code container in the
  memory descriptor.

* `cpumask_var_t cpu_vm_mask_var` - __TBD__

* `mm_context_t context` - Architecture-specific MMU context, though x86-64
  doesn't need to store context data here, it's used to store various x86-64
  specific data - [mm_context_t][mm_context_t] contains [VDSO][vdso], x86-64
  RDPMC performance counters, a flag indicating whether 32-bit emulation mode is
  available, and a per-process [local descriptor table][ldt] for running 16-bit
  segmented code in e.g. DOSBox or Wine.

* `unsigned long flags` - __TBD__

* `struct core_state *core_state` - __TBD__

* `spinlock_t ioctx_lock` (only if `CONFIG_AIO`) - __TBD__

* `struct kioctx_table *ioctx_table` (only if `CONFIG_AIO`) - __TBD__

* `struct task_struct *owner` (only if `CONFIG_MEMCG`) - The canonical owner of
  this memory descriptor.

* `struct file *exe_file` - A reference to the file that started this process,
  for use in the `/proc/<pid>/exe` symlink.

* `struct mmu_notifier_mm *mmu_notifier_mm` (only if `CONFIG_MMU_NOTIFIER`) -
  __TBD__

* `bool tlb_flush_pending` - __TBD__

* `struct uprobes_state uprobes_state` - __TBD__

* `atomic_long_t hugetlb_usage` (only if `CONFIG_HUGETLB_PAGE`) - __TBD__

### Initialisation

* The first [struct mm_struct][mm_struct] in the system is the statically
  allocated [init_mm][init_mm].

* On a fork, the parent [struct mm_struct][mm_struct] is copied via
  [copy_mm()][copy_mm], which also subsequently copies all page tables unless
  it's a kernel thread which borrows the previous [struct mm_struct][mm_struct],
  or the fork is invoked with `CLONE_VM` in which case the
  [struct mm_struct][mm_struct] is shared.

* `copy_mm()` invokes [dup_mm()][dup_mm] to do the heavy lifting of allocating
  the new [struct mm_struct][mm_struct] and subsequently duplicating the
  parent's page tables which is performed by [dup_mmap()][dup_mmap].

* On an `exec` an entirely new [struct mm_struct][mm_struct] is allocated and
  zeroed via [mm_alloc()][mm_alloc].

* Both of these functions use [allocate_mm()][allocate_mm] to perform the
  allocation via the [slab][slab] allocator. They also both initialise
  process-specific fields are initialised via [mm_init()][mm_init].

### Freeing

* A [struct mm_struct][mm_struct] is ultimately freed via [free_mm()][free_mm]
  which frees the object from the slab allocator.

* `free_mm()` is invoked either when [mm_init()][mm_init] fails or in
  [__mmdrop()][__mmdrop] which is called by [mmdrop()][mmdrop] when the
  descriptor's reference count, `mm_count`, is reduced to zero.

## Virtual Memory Areas

```
     |------------------|      |------------------|      |------------------|
     | struct mm_struct | .... | struct mm_struct | .... | struct mm_struct |
     |------------------|      |------------------|      |------------------|
                                  mmap | ^
                                       | |
                 /---------------------/ \-----------------------\
                 |                                               |
                 v                                               | vm_mm
     |-----------------------|       vm_next         |-----------------------| vm_next
     |                       | --------------------> |                       | - - ->
     | struct vm_area_struct |       vm_prev         | struct vm_area_struct | vm_prev
     |                       | <-------------------- |                       | <- - -
     |-----------------------|                       |-----------------------|
                 ^    | vm_file                                  | vm_ops
                 |    |                                          |
                 |    |                                          v
                 |    \-----------------\        |-----------------------------|
                 |                      |        | struct vm_operations_struct |
                 |                      v        |-----------------------------|
                 |               |-------------|
                 |               | struct file |
    Other VMAs   |               |-------------|
          .      |            f_mapping |
           .     |                      |
            .    |                      v
             \   |           |----------------------|
              \  |           | struct address_space |
               \ |           |----------------------|
                \|         i_mmap |     ^      | a_ops
(red-black tree) \----------------/     |      \---------------\
                                        |                      |
                                        |                      v
                                        |     |----------------------------------|
                                        |     |  struct address_space_operations |
                                        |     |----------------------------------|
                                        |
                       -  - - --/-------X-------\-- - -  -
                                |               |
                        mapping |               | mapping
                         |-------------| |-------------|
                         | struct page | | struct page |
                         |-------------| |-------------|
```

* The above diagram shows the relationship between the various process address
  space structures (far from exhaustively), which we'll get into below:

* Each process's virtual memory is additionally divided into non-overlapping
  regions (Virtual Memory Areas or 'VMA's) related by their purpose and
  protection state.

* This division exists because a process's virtual address space is necessarily
  sparse - only portions of the space are allocated at any one time, so it makes
  sense to track these regions.

* VMAs are represented by the [struct vm_area_struct][vm_area_struct] type.

* A [struct mm_struct][mm_struct]'s VMAs are stored both as a doubly-linked list
  and a [red-black tree][red-black], the head of the linked list being kept in
  the `mm_struct`'s `mmap` field, and previous/next nodes kept in the
  [struct vm_area_struct][vm_area_struct]'s `vm_prev` and `vm_next` fields,
  sorted in address order, and the red/black root in the `mm_struct`'s
  [struct rb_root][rb_root] `mm_rb` field, with the node kept in the
  `vm_area_struct`'s [struct rb_node][rb_node] `vm_rb` field.

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

* A list of a process's VMAs can be obtained via `/proc/<pid>/maps` or for
  additional usage statistics, `/proc/<pid>/smaps`.

* The [struct vm_area_struct][vm_area_struct] references its
  [struct mm_struct][mm_struct] back in turn via its `vm_mm` field.

* [struct vm_area_struct][vm_area_struct]'s `vm_ops` field references a
  [struct vm_operations_struct][vm_operations_struct] which contains function
  pointers which are invoked when certain events occur in the VMA:

```c
struct vm_operations_struct {
        void (*open)(struct vm_area_struct * area);
        void (*close)(struct vm_area_struct * area);
        int (*mremap)(struct vm_area_struct * area);
        int (*fault)(struct vm_area_struct *vma, struct vm_fault *vmf);
        int (*pmd_fault)(struct vm_area_struct *, unsigned long address,
                                                pmd_t *, unsigned int flags);
        void (*map_pages)(struct vm_area_struct *vma, struct vm_fault *vmf);

        /* notification that a previously read-only page is about to become
         * writable, if an error is returned it will cause a SIGBUS */
        int (*page_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);

        /* same as page_mkwrite when using VM_PFNMAP|VM_MIXEDMAP */
        int (*pfn_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);

        /* called by access_process_vm when get_user_pages() fails, typically
         * for use by special VMAs that can switch between memory and hardware
         */
        int (*access)(struct vm_area_struct *vma, unsigned long addr,
                      void *buf, int len, int write);

        /* Called by the /proc/PID/maps code to ask the vma whether it
         * has a special name.  Returning non-NULL will also cause this
         * vma to be dumped unconditionally. */
        const char *(*name)(struct vm_area_struct *vma);

        /*
         * set_policy() op must add a reference to any non-NULL @new mempolicy
         * to hold the policy upon return.  Caller should pass NULL @new to
         * remove a policy and fall back to surrounding context--i.e. do not
         * install a MPOL_DEFAULT policy, nor the task or system default
         * mempolicy.
         */
        int (*set_policy)(struct vm_area_struct *vma, struct mempolicy *new);

        /*
         * get_policy() op must add reference [mpol_get()] to any policy at
         * (vma,addr) marked as MPOL_SHARED.  The shared policy infrastructure
         * in mm/mempolicy.c will do this automatically.
         * get_policy() must NOT add a ref if the policy at (vma,addr) is not
         * marked as MPOL_SHARED. vma policies are protected by the mmap_sem.
         * If no [shared/vma] mempolicy exists at the addr, get_policy() op
         * must return NULL--i.e., do not "fallback" to task or system default
         * policy.
         */
        struct mempolicy *(*get_policy)(struct vm_area_struct *vma,
                                        unsigned long addr);
        /*
         * Called by vm_normal_page() for special PTEs to find the
         * page for @addr.  This is useful if the default behavior
         * (using pte_page()) would not find the correct page.
         */
        struct page *(*find_special_page)(struct vm_area_struct *vma,
                                          unsigned long addr);
};
```

* Of most note here are `fault()`, which is called when a
  [page fault][page-fault] occurs and `map_pages()` which maps a specified range
  of addresses. These both use [struct vm_fault][vm_fault] to parameterise their
  operation and to place certain out fields:

```c
struct vm_fault {
        unsigned int flags;             /* FAULT_FLAG_xxx flags */
        gfp_t gfp_mask;                 /* gfp mask to be used for allocations */
        pgoff_t pgoff;                  /* Logical page offset based on vma */
        void __user *virtual_address;   /* Faulting virtual address */

        struct page *cow_page;          /* Handler may choose to COW */
        struct page *page;              /* ->fault handlers should return a
                                         * page here, unless VM_FAULT_NOPAGE
                                         * is set (which is also implied by
                                         * VM_FAULT_ERROR).
                                         */
        /* for ->map_pages() only */
        pgoff_t max_pgoff;              /* map pages for offset from pgoff till
                                         * max_pgoff inclusive */
        pte_t *pte;                     /* pte entry associated with ->pgoff */
};
```

* Most files that are mapped to memory will not have any need for custom
  handling and hence will use the [generic_file_vm_ops][generic_file_vm_ops]
  [struct vm_operations_struct][vm_operations_struct]:

```c
const struct vm_operations_struct generic_file_vm_ops = {
        .fault          = filemap_fault,
        .map_pages      = filemap_map_pages,
        .page_mkwrite   = filemap_page_mkwrite,
};
```

* This invokes [filemap_fault()][filemap_fault] on a [page fault][page-fault],
  [filemap_map_pages()][filemap_map_pages] when pages are to be mapped, and
  [filemap_page_mkwrite()][filemap_page_mkwrite] to handle the case where a
  read-only page is about to be made writeable.

* If a memory region is mapped to a file, its `vm_file` field will be non-NULL
  and point to a [struct file][file] which describes the file which is mapped.

* The [struct file][file] in turn references a
  [struct address_space][address_space] via `f_mapping` which contains the data
  needed to track memory mapped to the file it describes:

```c
struct address_space {
        struct inode            *host;          /* owner: inode, block_device */
        struct radix_tree_root  page_tree;      /* radix tree of all pages */
        spinlock_t              tree_lock;      /* and lock protecting it */
        atomic_t                i_mmap_writable;/* count VM_SHARED mappings */
        struct rb_root          i_mmap;         /* tree of private and shared mappings */
        struct rw_semaphore     i_mmap_rwsem;   /* protect tree, count, list */
        /* Protected by tree_lock together with the radix tree */
        unsigned long           nrpages;        /* number of total pages */
        /* number of shadow or DAX exceptional entries */
        unsigned long           nrexceptional;
        pgoff_t                 writeback_index;/* writeback starts here */
        const struct address_space_operations *a_ops;   /* methods */
        unsigned long           flags;          /* error bits/gfp mask */
        spinlock_t              private_lock;   /* for use by the address_space */
        struct list_head        private_list;   /* ditto */
        void                    *private_data;  /* ditto */
}
```

* Note the [struct rb_root][rb_root] field `i_mmap` - this provides a
  [red-black][red-black] tree root listing private and shared VMAs which map the
  file, including the VMA that references the address space via
  `vm_file->f_mapping`.

* The operations a [struct address_space][address_space] needs to perform are
  provided via its [struct address_space_operations][address_space_operations]
  `a_ops` field, for example reading/writing pages from/to the file, etc.:

```c
struct address_space_operations {
        int (*writepage)(struct page *page, struct writeback_control *wbc);
        int (*readpage)(struct file *, struct page *);

        /* Write back some dirty pages from this mapping. */
        int (*writepages)(struct address_space *, struct writeback_control *);

        /* Set a page dirty.  Return true if this dirtied it */
        int (*set_page_dirty)(struct page *page);

        int (*readpages)(struct file *filp, struct address_space *mapping,
                        struct list_head *pages, unsigned nr_pages);

        int (*write_begin)(struct file *, struct address_space *mapping,
                                loff_t pos, unsigned len, unsigned flags,
                                struct page **pagep, void **fsdata);
        int (*write_end)(struct file *, struct address_space *mapping,
                                loff_t pos, unsigned len, unsigned copied,
                                struct page *page, void *fsdata);

        /* Unfortunately this kludge is needed for FIBMAP. Don't use it */
        sector_t (*bmap)(struct address_space *, sector_t);
        void (*invalidatepage) (struct page *, unsigned int, unsigned int);
        int (*releasepage) (struct page *, gfp_t);
        void (*freepage)(struct page *);
        ssize_t (*direct_IO)(struct kiocb *, struct iov_iter *iter, loff_t offset);
        /*
         * migrate the contents of a page to the specified target. If
         * migrate_mode is MIGRATE_ASYNC, it must not block.
         */
        int (*migratepage) (struct address_space *,
                        struct page *, struct page *, enum migrate_mode);
        int (*launder_page) (struct page *);
        int (*is_partially_uptodate) (struct page *, unsigned long,
                                        unsigned long);
        void (*is_dirty_writeback) (struct page *, bool *, bool *);
        int (*error_remove_page)(struct address_space *, struct page *);

        /* swapfile support */
        int (*swap_activate)(struct swap_info_struct *sis, struct file *file,
                                sector_t *span);
        void (*swap_deactivate)(struct file *file);
};
```

* Non-anonymously mapped [struct page][page]'s which represent the underlying
  physical pages of memory mapped to a [struct address_space][address_space]
  reference it via their `mapping` field (if & only if the low bit of the
  `mapping` field is clear.)

## Page Faulting

* A [page fault][page-fault] occurs when either a virtual memory address is not
  mapped, or it is and no physical page is actually mapped to the address
  (i.e. [pte_present()][pte_present] returns 0 on the mapping's PTE), or a write
  is attempted on a read-only mapping (i.e. [pte_write()][pte_write] returns 0
  on the mapping's PTE) or when userland tries to access a kernel mapping.

* This isn't necessary an error, in fact modern operating systems (including
  linux, naturally) use page faulting for a number of different mechanisms:

1. [Swap][swap] - 'Swapped out' memory is data that has been written to the disk
   so that RAM can be used for other purposes. The memory is marked non-present,
   then when a fault occurs the data can be read from disk and placed in memory
   for the referencing process to access.

2. [Demand paging][demand-paging] - Under demand paging, physical pages are not
   actually mapped until they are used. This allows for a far more efficient
   means of allocation of memory compared to the case where memory must be fully
   allocated as soon as it is requested - as soon as the memory is actually used
   in some way, a page fault occurs and the kernel allocates the memory.

3. [Copy-on-write semantics][copy-on-write] - Since page faults occur when a
   read-only page of memory is attempted to be written to, this can be used to
   significantly improve the efficiency of a [fork][fork]. Rather than copying
   the allocated pages of the parent process, the pages can be marked read-only
   in both parent and child, and as soon as one writes to a page the arising
   fault triggers a copy of the page. Since most process's pages of memory are
   only read, this hugely increases the efficiency of forks.

### Handling Page Faults

* Under x86, the kernel can specify an interrupt handler for page faults which
  is invoked by the CPU when one occurs. When the handler function is invoked,
  an error code describing the cause of the fault is pushed to the stack and the
  `cr2` control register is set to the linear address that caused the fault.

* The linux page fault handler is [do_page_fault()][do_page_fault]. It retrieves
  the error code and address before handing off the heavy lifting to
  [__do_page_fault()][__do_page_fault].

* [__do_page_fault()][__do_page_fault] performs a number of checks to handle the
  error cases, before handing off to the non-arch specific
  [handle_mm_fault()][handle_mm_fault] if the fault looks
  valid. Diagrammatically (simplifying the process somewhat, ignoring
  [kmemcheck][kmemcheck] and [kprobes][kprobes], some special case fault
  support, and vsyscall emulation - better to look at the code for those cases):

```
                                       -------------------
                                       |   CPU invokes   |
                                       | do_page_fault() |
                                       -------------------
                                                |
                                                v
                                    ------------------------
                                    | Retrieve address and |
                                    | error code then call |
                                    |   __do_page_fault()  |
                                    ------------------------
                                                |
                                                v
                                   /------------------------\
                                  /  Did we fault in kernel  \---\
                                  \          space?          /   | Yes
                                   \------------------------/    |
                                      | No                       v
                                      v                 /-----------------\ Yes
   ---------               Yes /-------------\         /  vmalloc_fault()  \----\
   | OOPS! |<-----------------/ Reserved bits \        \      succeed?     /    |
   ---------                  \   modified?   /         \-----------------/     |
                               \-------------/                   | No           |
                                      | No                       v              |
                                      v                    /-----------\ Yes    |
                          Yes /--------------\            /  Was fault  \-------\
   /-------------------------/      SMAP      \           \  spurious?  /       |
   |                         \   violation?   /            \-----------/        |
   |                          \--------------/                   | No           |
   |                                  | No                       |              |
   /----------------------------------(--------------------------/              |
   |                                  v                                         |
   |                  Yes /-----------------------\                             |
   /---------------------/  Fault handler disabled \                            |
   |                     \    or !current->mm?     /                            |
   |                      \-----------------------/                             |
   |                                  | No                                      |
   |                                  v                                         |
   |                         --------------------                               |
   |                         | Find nearest VMA |<--------\                     |
   |                         --------------------         |                     |
   |                                  |                   |                     |
   |                                  v                   |                     |
   |  No /-------------\  No /----------------\           |                     |
   /----/  Is region a  \<--/     Does VMA     \          | No                  |
   |    \     stack?    /   \ contain address? /    /------------\ Yes          |
   |     \-------------/     \----------------/    / Fatal signal \--\          |
   |            | Yes                 | Yes        \   pending?   /  |          |
   |            v                     |             \------------/   |          |
   | Yes /--------------\             |                   ^          |          |
   /----/  Is address <  \            |                   |          |          |
   |    \ stack pointer? /            |                   |          |          |
   |     \--------------/             |                   |          |          |
   |            | No                  |                   |          |          |
   |            v                     v             --------------   |          |
   | No /---------------\ Yes   /-----------\       | Mark retry |   |          |
   /---/  expand_stack() \---> / Permissions \      | disallowed |   |          |
   |   \    succeed?     /     \     OK?     /      --------------   |          |
   |    \---------------/       \-----------/             ^          |          |
   |                         No   |   | Yes               |          |          |
   /------------------------------/   |                   |          |          |
   |                                  v                   |          |          |
   |                        ---------------------         | Yes      |          |
   |                        |        Call       |     /--------\ No  |          |
   |                        | handle_mm_fault() |    /  Retry   \----\          |
   |                        ---------------------    \ allowed? /    |          |
   |                                  |               \--------/     |          |
   |                                  v                   ^          |          |
   |                          /---------------\  Yes      |          v          |
   |                         / Return Value &  \----------/     /---------\ Yes |
   |                         \ VM_FAULT_RETRY? /               /   From    \----\
   |                          \---------------/                \ userland? /    |
   |                                  | No                      \---------/     |
   |                                  v                              | No       |
   |  ------------------  Yes /---------------\                      |          |
   |  | mm_fault_error |<----/ Return value &  \                     |          |
   |  ------------------     \ VM_FAULT_ERROR? /                     |          |
   |           |              \---------------/                      |          |
   |           |                      | No                           |          |
   |           |                      v                              |          |
   |           |              ******************                     |          |
   |           |              * FAULT COMPLETE *<--------------------)----------/
   |           |              ******************                     |
   |           |                                                     |
   |           \---------------\                                     |
   |                           |                                     |
   |                           v                                     |
   |                /--------------------\ Yes                       |
   |               / Fatal signal pending \--------------------------\
   |               \   in kernel mode?    /                          |
   |                \--------------------/                           |
   |                           | No                                  |
   |                           v                                     |
   |                   /--------------\ Yes     /---------\ Yes      |
   |                  /     fault &    \------>/ In kernel \---------\
   |                  \  VM_FAULT_OOM? /       \   mode?   /         |
   |                   \--------------/         \---------/          |
   |                           | No                  | No            |
   |                           v                     v               |
   |  ----------  Yes /-----------------\     --------------         |
   |  |  Send  |<----/ fault & VM_FAULT_ \    | Invoke OOM |         |
   |  | SIGBUS |     \ SIGBUS/HWPOISON*? /    |   killer   |         |
   |  ----------      \-----------------/     --------------         |
   |                           | No                                  |
   |                           v                                     |
   |                  /-----------------\ No  ----------             |
   |                 /      fault &      \--->| BUG()! |             |
   |                 \  VM_FAULT_SIGSEGV /    ----------             |
   |                  \-----------------/                            |
   |                           | Yes                                 |
   |                           v                                     v
   |             -----------------------------               -----------------
   \------------>| __bad_area_no_semaphore() |       /------>|  no_context() |
                 -----------------------------       |       -----------------
                               |                     |               |
                               v                     |               v
                  /------------------------\ Yes     |     /------------------\ No
                 /  Did we fault in kernel  \--------/    / Is there a kernel  \--\
                 \          space?          /             \ exception handler? /  |
                  \------------------------/               \------------------/   |
                               | No                                  | Yes        |
                               v                                     v            |
                    /-------------------\ Yes               ------------------    |
                   / Attempted access of \-----------\      | Call exception |    |
                   \    kernel memory?   /           |      |     handler    |    |
                    \-------------------/            |      ------------------    |
                               | No                  |                            |
                               v                     v                            |
                       ----------------     -------------------   ---------       |
                       | Send SIGSEGV |<--- | Mark protection |   | OOPS! |<------/
                       ----------------     |      fault      |   ---------
                                            -------------------
```

#### Kernel faults

* Page faults in the kernel are not permitted, except for the case of `vmalloc`
  memory (will discuss this in a later section), which is used to allow for
  virtually contiguous memory.

#### Page Fault Error Code

* The error code specified by x86 consists of a bitfield of
  [enum x86_pf_error_code][x86_pf_error_code] providing information on the cause
  of the page fault:

```c
/*
 * Page fault error code bits:
 *
 *   bit 0 ==    0: no page found       1: protection fault
 *   bit 1 ==    0: read access         1: write access
 *   bit 2 ==    0: kernel-mode access  1: user-mode access
 *   bit 3 ==                           1: use of reserved bit detected
 *   bit 4 ==                           1: fault was an instruction fetch
 *   bit 5 ==                           1: protection keys block access
 */
enum x86_pf_error_code {

        PF_PROT         =               1 << 0,
        PF_WRITE        =               1 << 1,
        PF_USER         =               1 << 2,
        PF_RSVD         =               1 << 3,
        PF_INSTR        =               1 << 4,
        PF_PK           =               1 << 5,
};
```

* This is used by [__do_page_fault()][__do_page_fault] and the functions it
  invokes to handle cases correctly. Note that it enables the code to determine
  whether the access was made from userland or kernel mode, and given we know
  the address we can determine whether it is a kernel address or userland one.

#### Generic Page Table Fault Handling

* As shown in the flow chart above, an initial check is made against a number of
  error states, before the generic (i.e. non-arch specific)
  [handle_mm_fault()][handle_mm_fault] is called.

* [__handle_mm_fault()][__handle_mm_fault] does the heavy lifting, allocating
  page tables as necessary.

* It's interesting to note here that in fact page table mappings might not exist
  for _valid_ addresses. Looking back to the flow chart, we can see that we
  don't differentiate between a page fault caused by invalid virtual address
  (i.e. no page table mappings for that address) or one where the present bit is
  not set in the PTE.

* Instead, we use the VMA to determine if the address is valid (with a special
  case for stacks.) This means that large blocks of memory can not only fault-in
  the physical pages of memory when requested, but also the page tables required
  to map those pages.

* In practice, it seems that multi-page allocations result in the first and last
  pages being mapped and those in the middle not being. Experiment with the
  [multi-page alloc][multi-page-alloc] sample code and the
  [pagetables hack][pagetables-hack], both from the sister repo
  [linux-vm-hacks][linux-vm-hacks] to explore this on a local linux system.

* Physical page allocation is handled via
  [handle_pte_fault()][handle_pte_fault]. Firstly, the function needs to handle
  cases where the PTE is not present:

1. If the PTE is empty and the VMA is anonymous i.e. not mapping a file,
   [do_anonymous_page()][do_anonymous_page] handles allocating an anonymous page.

2. If the PTE is empty and the VMA is not anonymous, i.e. mapping a file,
   [do_fault()][do_fault] handles allocating the page.

3. Finally, if the PTE is not present but also not empty, then
   [do_swap_page()][do_swap_page] is invoked to handle the swapping.

* In each of the above cases, the relevant function is returned and the rest of
  the function does not run.

#### Page Fault Types

* Page faults are divided into 3 types - minor/soft, major/hard and error, the
  latter two cases represented by [VM_FAULT_MAJOR][VM_FAULT_MAJOR] and
  [VM_FAULT_ERROR][VM_FAULT_ERROR] respectively (`VM_FAULT_ERROR` is a bitmask
  of error states.)

* Minor (or soft) page faults are those where either memory simply needs to be
  faulted in or a [CoW][copy-on-write] copy needs to be made.

* Major (or hard) page faults are those that require I/O - i.e. a page needs to be
  swapped back into memory, or a file mapping needs to be flushed or read from.

[MAX_GAP]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L55
[MIN_GAP]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L54
[PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L35
[TASK_RSS_EVENTS_THRESH]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L165
[TASK_SIZE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L755
[TASK_UNMAPPED_BASE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L785
[VM_FAULT_ERROR]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1101
[VM_FAULT_MAJOR]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1088
[__PAGE_OFFSET]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L35
[__START_KERNEL_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64_types.h#L37
[__do_page_fault]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L1169
[__handle_mm_fault]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3412
[__mmdrop]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L691
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[__phys_addr_nodebug]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L12
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[address_space]:https://github.com/torvalds/linux/blob/v4.6/include/linux/fs.h#L430
[address_space_operations]:https://github.com/torvalds/linux/blob/v4.6/include/linux/fs.h#L372
[allocate_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L566
[arch_get_unmapped_area]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/sys_x86_64.c#L125
[arch_get_unmapped_area_topdown]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/sys_x86_64.c#L163
[arch_mmap_rnd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L68
[arch_pick_mmap_layout]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L100
[aslr]:https://en.wikipedia.org/wiki/Address_space_layout_randomization
[atomic_inc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/atomic.h#L89
[auxv]:http://articles.manugarg.com/aboutelfauxiliaryvectors
[copy-on-write]:https://en.wikipedia.org/wiki/Copy-on-write
[copy_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L958
[create_elf_tables]:https://github.com/torvalds/linux/blob/v4.6/fs/binfmt_elf.c#L150
[demand-paging]:https://en.wikipedia.org/wiki/Demand_paging
[do_anonymous_page]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L2729
[do_fault]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3182
[do_page_fault]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L1399
[do_swap_page]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L2511
[dup_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L923
[dup_mmap]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L408
[exit_mmap]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2719
[file]:https://github.com/torvalds/linux/blob/v4.6/include/linux/fs.h#L873
[filemap_fault]:https://github.com/torvalds/linux/blob/v4.6/mm/filemap.c#L2013
[filemap_map_pages]:https://github.com/torvalds/linux/blob/v4.6/mm/filemap.c#L2134
[filemap_page_mkwrite]:https://github.com/torvalds/linux/blob/v4.6/mm/filemap.c#L2207
[fork]:https://en.wikipedia.org/wiki/Fork_(system_call)
[free_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L567
[generic_file_vm_ops]:https://github.com/torvalds/linux/blob/v4.6/mm/filemap.c#L2234
[handle_mm_fault]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3501
[handle_pte_fault]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3345
[init_mm]:https://github.com/torvalds/linux/blob/v4.6/mm/init-mm.c#L16
[kdump-paper]:https://www.kernel.org/doc/ols/2007/ols2007v1-pages-167-178.pdf
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[kmemcheck]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kmemcheck.txt
[kprobes]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kprobes.txt
[ldt]:https://en.wikipedia.org/wiki/Global_Descriptor_Table#Local_Descriptor_Table
[mm_alloc]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L674
[mm_context_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/mmu.h#L11
[mm_init]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L598
[mm_rss_stat]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L385
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mmap_is_legacy]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L57
[mmdrop]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L2613
[mmlist_lock]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L564
[mmput]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L705
[page-fault]:https://en.wikipedia.org/wiki/Page_fault
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[phys_base-fixup]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L140
[phys_base]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L520
[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[pinned_vm-commit]:https://github.com/torvalds/linux/commit/bc3e53f682d93df677dbd5006a404722b3adfe18
[pte_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L491
[pte_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L131
[rb_node]:https://github.com/torvalds/linux/blob/v4.6/include/linux/rbtree.h#L36
[rb_root]:https://github.com/torvalds/linux/blob/v4.6/include/linux/rbtree.h#L43
[red-black]:https://en.wikipedia.org/wiki/Red%E2%80%93black_tree
[rss]:https://en.wikipedia.org/wiki/Resident_set_size
[split-page-table-lock]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/split_page_table_lock
[swap]:https://en.wikipedia.org/wiki/Paging
[task_mem]:https://github.com/torvalds/linux/blob/v4.6/fs/proc/task_mmu.c#L24
[task_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L1394
[unmap_vmas]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L1350
[vdso]:https://en.wikipedia.org/wiki/VDSO
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[vm_area_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L294
[vm_fault]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L290
[vm_operations_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L313
[vm_pinned-thread]:https://marc.info/?t=142540464700001&r=1&w=1
[vma-cache]:https://lwn.net/Articles/589475/
[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
[x86_pf_error_code]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L40
[linux_binfmt]:https://github.com/torvalds/linux/blob/v4.6/include/linux/binfmts.h#L74

[linux-vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks
[multi-page-alloc]:https://github.com/lorenzo-stoakes/linux-vm-hacks/blob/master/experiments/multi_page_alloc.c
[page-tables]:./page-tables.md
[pagetables-hack]:https://github.com/lorenzo-stoakes/linux-vm-hacks/tree/master/pagetables
[slab]:./slub.md
