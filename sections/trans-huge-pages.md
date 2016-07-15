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
  pages are 1GiB.

* Using larger pages makes sense for tasks which allocate larger blocks of
  memory because [TLB][tlb] resources are scarce and TLB misses are
  expensive. Because the physical page size is larger, each TLB entry now maps
  512 times as much memory (2MiB/4KiB = 512.)

* In addition page faulting is reduced by a factor of 512, however this is less
  impactful as this speed up occurs only once in the lifetime of the memory
  region.

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
