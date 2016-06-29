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

## TODO high

* Discuss split page table locks in page-tables.md.

## TODO general

* Cover `switch_mm()` etc.

* Fixup link to vmalloc in [functions](./funcs.md) page from being a link to
  http://www.makelinux.net/books/lkd2/ch11lev1sec5 to a local description once
  it's written.

* Cover [mem_section][mem_section]/memory sections. Seems like the _physical_
  memory map is divided into these 'sections' which determine whether a given
  block of memory (seem to be in 128MiB blocks on x86-64) is actually backed by
  RAM or not :]. Also, go into detail about the sparse memory model for x86-64.

* [Page Attribute Table (PAT)][pat]? [Memory type range register (MTRR)][mtrr]?
  Some fine-grained caching stuff going on there, cover. Also PCD (Page-level
  Cache Disable) and PWT (Page-level Write-Through) flags. PCID seems related?

* Confirm that, by default, accessed and dirty flags are cleared in memory
  pages.

* Expand page flags list to be more exhaustive.

* Examine the wonders of `include/linux/pfn_t.h` - Seems to be a specific PFN
  type with flag bitfields added in.

* Check out the devmap stuff more thoroughly - descriptions like 'Determines if
  the PTE page pointed at by the specified PMD entry is part of a device
  mapping.' might not be valid. Surely a part of the
  [device mapper][device-mapper] functionality?

* Maybe add `_PAGE_HIDDEN` stuff - used by kmemcheck.

* The comment at line 93 in `arch/x86/include/asm/pgtable.h` says:

```c
/*
 * The following only work if pte_present() is true.
 * Undefined behaviour if not..
 */
```

* The 'following' suggests all functions below it, so the `pte_present()` caveat
  should probably be added to these flag functions (and possibly just all of
  them.)

* Look into why `pmd_mkclean()` and `pte_mkclean()` do not also clear the
  soft-dirty flag.

* Be consistent with discussion of 'user-defined' flags in `mk/clr` functions
  too.

* `preempt_disable/enable()` appears to simply call `barrier()` on x86 rather
  than actually doing anything to prevent preemption, why?!

* Weird that [flush_tlb_kernel_range()][flush_tlb_kernel_range] flushes
  individual pages given kernel mappings are marked `_PAGE_GLOBAL` (and afaict
  an `invlpg` instruction does not override this), seems mostly used by vmalloc
  stuff, maybe those pages aren't marked global?

* Talk about `rmap`s.

* `__handle_mm_fault()` seems to be a really core function.

* `struct per_cpu_pages` - is this used by modern kernels to obviate the need
  for specific 'quick lists' for allocating page table pages? Covered in what's
  new in linux 2.6 section of UtLVMM.

* What are `%esp` fixup stacks referenced in the [x86-64 memory map][x86-64-mm]?
  Presumably a means of handling 32-bit processes (given `esp` not `rsp`.)

* Cover ASLR (maybe mention DEP in passing, have already covered the NX bit.)

* `MODULES_END = 0xffffffffff000000` but the [x86-64 memory map][x86-64-mm] says
  they end at `0xffffffffff600000`. Why the difference?

* Cover anonymous memory.

* Cover spurious faults, e.g. [spurious_fault()][spurious_fault].

* Look at vsyscall emulation.

* Look at vmacache - https://lwn.net/Articles/589475/

* Trace through a fork/exec.

## Thoughts

* When I refer to PMDs (and even perhaps PUDs in the case of gigantic pages) as
  always referring to an underlying lower-level page table page, am I always
  correct in doing so? Since in huge page mode I _think_ a PMD with the PSE bit
  set actually refers to the final physical page, but perhaps I'm wrong. If a
  PMD does refer to a page this way, I will need to modify the function
  descriptions accordingly.

* What happens when a new kernel mapping is added in page tables? Are all shared
  mappings updated? Or kept to a single PUD maybe? Probably just pre-mapped. -
  `KERNEL_IMAGE_SIZE_DEFAULT` seems to indicate that the 512MiB kernel is
  pre-mapped.

## Interesting functions

* [free_pgd_range()][free_pgd_range].

* [vm_normal_page()][vm_normal_page] - has some discussion about 'special' PTEs,
  worth investigating.

[device-mapper]:https://en.wikipedia.org/wiki/Device_mapper
[flush_tlb_kernel_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L296
[free_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L473
[mem_section]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1040
[mtrr]:https://en.wikipedia.org/wiki/Memory_type_range_register
[pat]:https://en.wikipedia.org/wiki/Page_attribute_table
[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
[spurious_fault]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L1045
[vm_normal_page]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L742
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
