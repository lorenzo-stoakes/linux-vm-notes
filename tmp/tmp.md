## vma_merge()

determines if a mapping request can be merged with its predecessor or successor.

### Called from:

(Usually, there are calls from other places but for less standard things):

[copy_vma()][copy_vma]
[do_brk()][do_brk]
[mmap_region()][mmap_region]

[copy_vma]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2806
[do_brk]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2621
[mmap_region]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L1424
[vma_merge]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L934
