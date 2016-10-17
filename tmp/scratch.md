# Scratch Pad

This file contains notes that have not yet been assigned to any particular
section.

## Free Memory Areas

* [show_free_areas()][show_free_areas] outputs a summary of the state of each
  populated zone for a node (in UMA, a single node) to the kernel ring buffer
  (i.e. `dmesg`.) Some sample output from an OOM killer invocation:

```
active_anon:467951 inactive_anon:17639 isolated_anon:0
 active_file:451743 inactive_file:273949 isolated_file:0
 unevictable:8 dirty:6375 writeback:0 unstable:0
 slab_reclaimable:655758 slab_unreclaimable:18473
 mapped:107497 shmem:20748 pagetables:6394 bounce:0
 free:32589 free_pcp:188 free_cma:0
Node 0 DMA free:15644kB min:136kB low:168kB high:200kB active_anon:0kB inactive_anon:0kB active_file:0kB inactive_file:0kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:15984kB managed:15900kB mlocked:0kB dirty:0kB writeback:0kB mapped:0kB shmem:0kB slab_reclaimable:0kB slab_unreclaimable:0kB kernel_stack:0kB pagetables:0kB unstable:0kB bounce:0kB free_pcp:0kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:0 all_unreclaimable? yes
lowmem_reserve[]: 0 3196 7658 7658
Node 0 DMA32 free:64452kB min:28148kB low:35184kB high:42220kB active_anon:853640kB inactive_anon:31568kB active_file:565352kB inactive_file:536548kB unevictable:32kB isolated(anon):0kB isolated(file):0kB present:3617864kB managed:3280096kB mlocked:32kB dirty:10244kB writeback:0kB mapped:162416kB shmem:36576kB slab_reclaimable:1122360kB slab_unreclaimable:28104kB kernel_stack:2896kB pagetables:11972kB unstable:0kB bounce:0kB free_pcp:0kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:220 all_unreclaimable? no
lowmem_reserve[]: 0 0 4462 4462
Node 0 Normal free:50260kB min:39296kB low:49120kB high:58944kB active_anon:1018164kB inactive_anon:38988kB active_file:1241620kB inactive_file:559248kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:4702208kB managed:4569312kB mlocked:0kB dirty:15256kB writeback:100kB mapped:267572kB shmem:46416kB slab_reclaimable:1500672kB slab_unreclaimable:45788kB kernel_stack:5616kB pagetables:13604kB unstable:0kB bounce:0kB free_pcp:740kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:0 all_unreclaimable? no
lowmem_reserve[]: 0 0 0 0
Node 0 DMA: 1*4kB (U) 1*8kB (U) 1*16kB (U) 0*32kB 2*64kB (U) 1*128kB (U) 0*256kB 0*512kB 1*1024kB (U) 1*2048kB (M) 3*4096kB (M) = 15644kB
Node 0 DMA32: 4950*4kB (UME) 5344*8kB (UME) 158*16kB (UME) 0*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 65080kB
Node 0 Normal: 9878*4kB (UME) 1286*8kB (UME) 5*16kB (UM) 0*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 49880kB
Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=1048576kB
Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=2048kB
746391 total pagecache pages
0 pages in swap cache
Swap cache stats: add 0, delete 0, find 0/0
Free swap  = 0kB
Total swap = 0kB
```

* The order-0 (4KiB), order-1 (8KiB), etc. output here is an indication of the
  number of free pages of that order from the _physical page allocator_.

Each order of free memory pages are retrievable from:

zone->free_area[order]->nr_free;

Physical memory is really not so contiguous for a user process, e.g. a straightforward mmap gives - 0x1eca34000, 0x200025000, 0x212b36000, 0x177f8d000, etc.

[do_anonymous_page()][do_anonymous_page] uses
[alloc_zeroed_user_highpage_moveable()][alloc_zeroed_user_highpage_moveable] to
obtain a page for each fault so this makes sense.

Most allocations utilise order-0 pages (i.e. allocate page-by-page), but some
need more contiguous physical RAM, a classic example is a kernel stack which is order-1.

[alloc_zeroed_user_highpage_moveable]:https://github.com/torvalds/linux/blob/v4.6/include/linux/highmem.h#L180
[do_anonymous_page]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L2729
[show_free_areas]:https://github.com/torvalds/linux/blob/v4.6/mm/page_alloc.c#L3870
