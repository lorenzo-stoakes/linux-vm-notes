# Transparent Huge Pages

* [Transparent huge pages][transhuge] take advantage of the fact that the
  processor is able to use [page tables][page-tables] that through their flags
  specify that the final physical page they refer to is larger than the standard
  page size.

* It transparently converts processes' pages to huge pages where it makes sense
  to do so, without userland being aware or needing to take any action
  whatsoever.

* In order for transparent huge pages to be available,
  `CONFIG_TRANSPARENT_HUGEPAGE` has to be enabled.

* The mode in which transparent huge pages run is specified by
  `/sys/kernel/mm/transparent_hugepage/enabled`. This is set to one of 'always',
  'madvise', and 'never' - enabling unconditionally, enabling if memory is
  [madvise][madvise]d with `MADV_HUGEPAGE`, and disabling altogether,
  respectively.

## Huge Pages

* On x86 the standard page size is 4KiB, huge pages are 2MiB and 'gigantic'
  pages are 1GiB. For the time being we consider huge pages only (1GiB pages are
  pretty specialist :)

* Using larger pages makes sense for tasks which allocate larger blocks of
  memory because [TLB][tlb] resources are scarce and TLB misses are
  expensive. Because the physical page size is larger, each TLB entry now maps
  512 times as much memory (2MiB/4KiB = 512.)

* In addition page faulting is reduced by a factor of 512, however this is less
  impactful as this speed up occurs only once in the lifetime of the memory
  region.

* Finally, TLB misses are less expensive as there is 1 less level of page tables
  to traverse since huge pages terminate at the PMD level.

### Page Tables

* When a page table entry at the PUD or PMD level has the `_PAGE_PSE` flag set
  this indicates that it refers to a gigantic or huge page, respectively. In
  x86-64, a gigantic page is 1GiB and a huge page 2MiB in size.

* Looking back at the [page tables][page-tables] section, we can see that the
  'shifts' (i.e. the offset in bits each page table offset sits at in the
  virtual address) indicate how many bits each page table level encompasses:

```c
#define PGDIR_SHIFT     39
#define PUD_SHIFT       30
#define PMD_SHIFT       21
#define PAGE_SHIFT      12
```

* So we can see that the PTE offset (i.e. `PAGE_SHIFT`) refers to 12 bits
  (`1<<12` = 4KiB) of data as there are 12 bits remaining the determine the
  offset into the physical page the PTE entry points at.

* Equally the PMD offset (i.e. `PMD_SHIFT`) refers to 21 bits (`1<<21` = 2MiB)
  of data as there are 21 bits remaining to determine the remaining PTE offset
  and physical page offset. The same goes for PUD at 1GiB and PGD at 512GiB.

* Typically, the page tables will be walked from PGD to PUD to PMD to PTE then
  finally the physical page. However, if the `_PAGE_PSE` flag is set at the PMD
  or PUD level then that entry is treated as the final one, i.e. the physical
  address and (other) flags are taken to refer to the final physical page, and
  the remaining bits in the virtual address are treated as an offset into this
  final physical page.

* This makes sense, as if we are to allow a physical page to be larger, we need
  more offset bits to be able to offset inside of it, and the virtual address
  then does not have sufficient 'space' to accommodate further page table
  offsets.

* The two forms of larger page size supported by x86-64 are gigantic and pages,
  1GiB and 2MiB respectively. For the time being we ignore gigantic pages, so it
  follows that huge pages are implemented by setting `_PAGE_PSE` at the PMD
  level, and therefore the remaining 21 bits are used to offset into the huge
  physical page.

## khugepaged

* The kernel thread which translates existing 4KiB pages to 2MiB huge pages is
  `khugepaged`, which executes [khugepaged()][khugepaged], which is invoked (and
  also stopped) via [start_stop_khugepaged()][start_stop_khugepaged].

* When run, the thread marks itself [cgroup freezable][cgroup-freezer] and sets
  its [niceness][nice] to maximum in order to run at lowest priority.

* It then enters a loop that runs while the kthread is not marked to stop in
  which it runs a scan via [khugepaged_do_scan()][khugepaged_do_scan] to perform
  the core of its operation.

* After the scan, the thread waits on the [khugepaged_wait][khugepaged_wait]
  [wait queue][wait-queue] via [khugepaged_wait_work()][khugepaged_wait_work].

* The number of pages that `khugepaged` scans for suitability to conversion to a
  huge page is determined by
  [khugepaged_pages_to_scan][khugepaged_pages_to_scan], which is tunable via
  `/sys/kernel/mm/transparent_hugepage/khugepaged/pages_to_scan` and initialised
  to 4096 on x86-64, equal to the number of pointers in a PMD page, `1 <<
  (PMD_SHIFT - PAGE_SHIFT)`, multiplied by 8.

* The number of additional smaller pages that can be allocated when collapsing
  smaller pages to large is determined by
  [khugepaged_max_ptes_none][khugepaged_max_ptes_none], which is tunable via
  `/sys/kernel/mm/transparent_hugepage/khugepaged/max_ptes_none`. This is
  initialised to 511 on x86-64, equal to the number of pointers in a PMD page,
  subtracting 1 - since 2MiB/4KiB = 512 this is equivalent to the additional
  space required to convert a single 4KiB page to a huge 2MiB one.

* The initialisation takes part in [hugepage_init()][hugepage_init].

[cgroup-freezer]:https://github.com/torvalds/linux/blob/v4.6/Documentation/cgroup-v1/freezer-subsystem.txt
[hugepage_init]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L662
[khugepaged]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L2816
[khugepaged_do_scan]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L2766
[khugepaged_pages_to_scan]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L86
[khugepaged_wait]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L95
[khugepaged_wait_work]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L2800
[khugepaged_max_ptes_none]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L101
[madvise]:http://man7.org/linux/man-pages/man2/madvise.2.html
[nice]:https://en.wikipedia.org/wiki/Nice_(Unix)
[start_stop_khugepaged]:https://github.com/torvalds/linux/blob/v4.6/mm/huge_memory.c#L179
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
[wait-queue]:http://www.makelinux.net/ldd3/chp-6-sect-2

[page-tables]:./page-tables.md
