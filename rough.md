# Rough Notes

__NOTE:__ These are rough notes intentionally not linked from elsewhere, and may
be full of errors, swearing and bullshit.

## General

* `pXX_large()` seems to duplicate `pXX_huge()`, though the latter seems to be
  in hugetlb context, and there are some variations, for example:

```
pud_large -> pse/present pud_huge -> pse
pmd_large -> pse         pmd_huge -> pse/not present
```

## 2.4.22 -> 4.6

* Seems like missing page levels is handled via
  [pgtable-nopmd.h][pgtable-nopmd.h] now.

* `CONFIG_PGTABLE_LEVELS` is used to determine number of pg table levels now.

## TODO next

### Page Tables

* Add page table allocation, modification, freeing functions, description.

* Add boot time description.

## TODO general

* Fixup link to vmalloc in [functions](./funcs.md) page from being a link to
  http://www.makelinux.net/books/lkd2/ch11lev1sec5 to a local description once
  it's written.

* Cover [mem_section][mem_section]/memory sections. Seems like the _physical_
  memory map is divided into these 'sections' which determine whether a given
  block of memory (seem to be in 128MiB blocks on x86-64) is actually backed by
  RAM or not :]. Also, go into detail about the sparse memory model for x86-64.

* [Page Attribute Table (PAT)][pat]? [Memory type range register (MTRR)][mtrr]?
  Some fine-grained caching stuff going on there, cover. Also PCD (Page-level
  Cache Disable) and PWT (Page-level Write-Through) flags.

* Carefully check whether the `_PAGE_PSE` flag in a PUD or PMD means that the
  entry refers to the final physical page, or whether it indicates the next
  level page table page is larger. I very strongly believe the former is the
  case, but need to investigate to be sure.

* Confirm that, by default, accessed and dirty flags are cleared in memory
  pages.

* Expand page flags list to be more exhaustive.

* Stupid question - does the global page flag cause a page mapping to remain in
  the TLB at _all_ times, not even being evicted due to not being used? I
  strongly suspect not, I suspect it just prevents flushes but not 'aging out'
  of the cache. But worth checking.

[PFN_PHYS]:https://github.com/torvalds/linux/blob/v4.6/include/linux/pfn.h#L20
[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
[mem_section]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1040
[pat]:https://en.wikipedia.org/wiki/Page_attribute_table
[mtrr]:https://en.wikipedia.org/wiki/Memory_type_range_register
