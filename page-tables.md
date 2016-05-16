# Page Tables

__Certainty Level:__ Moderate, very early in development, but this is pretty
fundamental stuff.

__Architecture:__ These notes assume an x86-64 UMA system unless otherwise
specified. [Linux 4.6][linux-4.6] is always targeted.

## Introduction

* [Virtual memory][virtual-memory], used by the majority of modern operating
  systems, uses the [MMU][mmu] hardware (for modern systems, read: CPU) of the
  system to allow [ring][ring]-0 privileged code to specify mappings between
  memory addresses and actual physical memory addresses via a CPU control
  register, rendering the actual addresses used by running code 'virtual'.

* The mappings are performed at the granularity of [page][page]s,
  i.e. contiguous blocks of memory of a set size. In x86-64 you can have page
  sizes of 4KiB, 2MiB and 1GiB, though typically 4KiB (2^12) will be used in
  most cases.

* The kernel switches out virtual memory mappings between userland processes to
  provide an abstraction layer and to isolate processes for each other, however
  this means that each process must store its own set of mappings. If the entire
  address space were mapped an x86-64 system would require 256GiB per process
  consisting overwhelming of null mappings.

* To avoid such egregious space requirements, mappings are subdivided into a
  'sparse' arrangement of [page table][page-table]s and the virtual address
  becomes a set of offsets into these tables along with an offset into the final
  physical page of memory (more explanation on what I mean by 'sparse' below.)

## Page Table Layout

* In linux there are 4 'levels' of page tables. A table consists of an array of
  entries of type `pXX_t`, wrapping a `pXXval_t`:

1. Page Global Directory (PGD) - [pgd_t][pgd_t]/[pgdval_t][pgdval_t].

2. Page Upper Directory (PUD) - [pud_t][pud_t]/[pudval_t][pudval_t].

3. Page Middle Directory (PMD) - [pmd_t][pmd_t]/[pmdval_t][pmdval_t].

4. Page Table Entry directory (PTE) - [pte_t][pte_t]/[pteval_t][pteval_t].

* These types are simply `typedef`'d wrappers around fundamental
  architecture-dependent types, gathering all the x86-64 types together:

```c
typedef unsigned long   pgdval_t;
typedef unsigned long   pudval_t;
typedef unsigned long   pmdval_t;
typedef unsigned long   pteval_t;

typedef struct { pgdval_t pgd; } pgd_t;
typedef struct { pudval_t pud; } pud_t;
typedef struct { pmdval_t pmd; } pmd_t;
typedef struct { pteval_t pte; } pte_t;
```

__NOTE:__ `p[gum]d_t` are defined in
[arch/x86/include/asm/pgtable_types.h][pgtable_types.h], `pte_t` and all the
`pXXval_t` types are defined in
[arch/x86/include/asm/pgtable_64_types.h][pgtable_64_types.h] - this shares as
much as possible between 32 and 64-bit x86.

* As mentioned above, a virtual address is simply a set of offsets into each of
  these tables. In the typical case of a 4KiB page size, each of the PGD, PUD,
  PMD, and PTE tables contain 512 pointers each, and since the word size is 8
  bytes, this means 4KiB of storage i.e. conveniently - each page table takes up
  a page of memory.

* The number of pointers available per table is defined in the `PTRS_PER_Pxx`
  preprocessor constant, i.e. [PTRS_PER_PGD][PTRS_PER_PGD],
  [PTRS_PER_PUD][PTRS_PER_PUD], [PTRS_PER_PMD][PTRS_PER_PMD], and
  [PTRS_PER_PTE][PTRS_PER_PTE].

* The number of bits a virtual address needs to be shifted right before being
  masked to obtain an index into a page table is defined by
  [PGDIR_SHIFT][PGDIR_SHIFT] for PGD, [PUD_SHIFT][PUD_SHIFT] for PUD,
  [PMD_SHIFT][PMD_SHIFT] for PMD and [PAGE_SHIFT][PAGE_SHIFT] for PTE (since at
  that point you just want to 'shift out' the physical page offset.)

* I'll go into the helper functions around this later, but once you've shifted
  an address, you can then simply mask it with `PTRS_PER_Pxx - 1` - this takes
  advantage of the fact that all values less than a power of 2 will be masked by
  the power minus 1 (e.g. `0b1000 - 1 = 0b0111`), so the `_SHIFT` and
  `PTRS_PER_` values are all you need to determine these indexes.

* Gathering all these values together for the typical 4KiB page size case:

```c
#define PGDIR_SHIFT     39
#define PMD_SHIFT       21
#define PUD_SHIFT       30

#define PTRS_PER_PGD    512
#define PTRS_PER_PUD    512
#define PTRS_PER_PMD    512
#define PTRS_PER_PTE    512
```

* And so we can now visualise what an given example address actually represents:

```
    6         5         4         3         2         1
4321098765432109876543210987654321098765432109876543210987654321
0000000000000000000000000000000000000000101100010111000000010000
[   RESERVED   ][  PGD  ][  PUD  ][  PMD  ][  PTE  ][  OFFSET  ]
                         |        |        |        |-PAGE_SHIFT
                         |        |        |-----------PMD_SHIFT
                         |        |--------------------PUD_SHIFT
                         |---------------------------PGDIR_SHIFT
```

### 64-bit Address Space

* In [current x86-64 implementations][x86-64-address-space] only the lower 48
  bits of an address are used - the remaining higher order bits must all be
  equal to the 47th bit, i.e. allowable addresses are 128TiB (47 bits) in the
  `0000000000000000` - `00007fffffffffff` range, and 128TiB (47 bits) in the
  `ffff800000000000` - `ffffffffffffffff` range - the address space is divided
  into upper and lower portions.

* In fact, reading the [x86-64 memory map doc][x86-64-mm], only 46 bits (64TiB)
  of RAM is supported - this makes sense, as it allows for the entire physical
  address space to be mapped into the kernel while leaving another 64TiB free
  for other kernel data.

* In linux, like most (if not all?) other operating systems, this provides a
  nice separation between kernel and user address space for free - keep kernel
  addresses in the upper portion and user addresses in the lower portion.

[linux-4.6]:https://github.com/torvalds/linux/blob/v4.6/

[virtual-memory]:https://en.wikipedia.org/wiki/Virtual_memory
[mmu]:https://en.wikipedia.org/wiki/Memory_management_unit
[ring]:https://en.wikipedia.org/wiki/Protection_ring
[page]:https://en.wikipedia.org/wiki/Page_(computer_memory)
[page-table]:https://en.wikipedia.org/wiki/Page_table
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pgdval_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pte_t]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[pgtable_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h
[pgtable_64_types.h]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[PGDIR_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L27
[PUD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L33
[PMD_SHIFT]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L40
[PAGE_SHIFT]:http://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_types.h#L8

[x86-64-address-space]:https://en.wikipedia.org/wiki/X86-64#VIRTUAL-ADDRESS-SPACE
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt

[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[pgtable-nopmd.h]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/pgtable-nopmd.h
