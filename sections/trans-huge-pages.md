# Transparent Huge Pages

* [Transparent huge pages][transhuge] take advantage of the fact that the
  processor is able to use [page tables][page-tables] that through their flags
  specify that the final physical page they refer to is larger than the standard
  page size.

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

[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt

[page-tables]:./page-tables.md
