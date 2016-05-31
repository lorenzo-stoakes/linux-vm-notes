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

## TODO

* Fixup link to vmalloc in [functions](./funcs.md) page from being a link to
  http://www.makelinux.net/books/lkd2/ch11lev1sec5 to a local description once
  it's written.

* Cover [mem_section][mem_section]/memory sections. Seems like the _physical_
  memory map is divided into these 'sections' which determine whether a given
  block of memory (seem to be in 128MiB blocks on x86-64) is actually backed by
  RAM or not :]. Also, go into detail about the sparse memory model for x86-64.

* [Page Attribute Table (PAT)][pat]? [Memory type range register (MTRR)][mtrr]?
  Some fine-grained caching stuff going on there, cover.

[PFN_PHYS]:https://github.com/torvalds/linux/blob/v4.6/include/linux/pfn.h#L20
[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
[mem_section]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1040
[pat]:https://en.wikipedia.org/wiki/Page_attribute_table
[mtrr]:https://en.wikipedia.org/wiki/Memory_type_range_register
