# Transparent Huge Pages

* [Transparent huge pages][transhuge] take advantage of the fact that the
  processor is able to use [page tables][page-tables] that through their flags
  specify that the final physical page they refer to is larger than the standard
  page size.

* This functionality transparently converts processes' pages to huge pages where
  it makes sense to do so, without userland being aware or needing to take any
  action whatsoever.

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

[madvise]:http://man7.org/linux/man-pages/man2/madvise.2.html
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt

[page-tables]:./page-tables.md
