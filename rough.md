# Rough Notes

__NOTE:__ These are rough notes intentionally not linked from elsewhere, and may
be full of errors, swearing and bullshit.

## General

* `pXX_pfn_mask()`, `__phys_to_pfn`, `__pfn_to_phys` / [PHYS_PFN()][PFN_PHYS],
  etc. - cover.

* `pXX_large()` seems to duplicate `pXX_huge()`, though the latter seems to be
  in hugetlb context, and there are some variations, for example:

```
pud_large -> pse/present pud_huge -> pse
pmd_large -> pse         pmd_huge -> pse/not present
```

* Need to consider `pgprot`s in general + how they relate to flags,
  e.g. `pte_flags()` returns a `pteval_t`.

## 2.4.22 -> 4.6

* Seems like missing page levels is handled via
  [pgtable-nopmd.h][pgtable-nopmd.h] now.

* `CONFIG_PGTABLE_LEVELS` is used to determine number of pg table levels now.

## TODO

## Big Blocks of Code

[PFN_PHYS]:https://github.com/torvalds/linux/blob/v4.6/include/linux/pfn.h#L20
[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
