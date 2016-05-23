# VM Functions

__Certainty Level:__ Moderate, am trying to check functions as best I can, but
might be making mistakes.

__Architecture:__ These notes assume an x86-64 UMA system unless otherwise
specified. [Linux 4.6][linux-4.6] is always targeted.

## Contents

### Address Translation

* [phys_to_virt()](#phys_to_virt) - Translates physical to virtual address.
* [virt_to_phys()](#virt_to_phys) - Translates virtual to physical address.
* [__va()](#__va) - Translates physical to virtual address.
* [__pa()](#__pa) - Translates virtual to physical address.

### Page Tables

#### Retrieving Page Table Entries

__NOTE:__ Confusingly, the `_offset` functions actually return a virtual address
for the page table entry associated with the specified virtual address.

* [pgd_offset()](#pgd_offset) - Gets virtual address of specified virtual
  address's PGD entry.
* [pud_offset()](#pud_offset) - Gets virtual address of specified virtual
  address's PUD entry.
* [pmd_offset()](#pmd_offset) - Gets virtual address of specified virtual
  address's PMD entry.
* [pte_offset_map()](#pte_offset_map) - Gets virtual address of specified
  virtual address's PTE entry (always mapped in x86-64.)
* [pte_offset_map_lock()](#pte_offset_map_lock) - Gets virtual address of
  specified virtual address's PTE entry then acquires PTE lock.

#### Retrieving Native Page Table Values

* [native_pgd_val()](#native_pgd_val) - Gets the native [pgdval_t][pgdval_t]
  associated with the specified PGD entry.
* [native_pud_val()](#native_pud_val) - Gets the native [pudval_t][pudval_t]
  associated with the specified PUD entry.
* [native_pmd_val()](#native_pmd_val) - Gets the native [pmdval_t][pmdval_t]
  associated with the specified PMD entry.
* [native_pte_val()](#native_pte_val) - Gets the native [pteval_t][pteval_t]
  associated with the specified PTE entry.
* [pgd_val()](#pgd_val) - Gets the [pgdval_t][pgdval_t] associated with the
  specified PGD entry, possibly paravirtualised.
* [pud_val()](#pud_val) - Gets the [pudval_t][pudval_t] associated with the
  specified PUD entry, possibly paravirtualised.
* [pmd_val()](#pmd_val) - Gets the [pmdval_t][pmdval_t] associated with the
  specified PMD entry, possibly paravirtualised.
* [pte_val()](#pte_val) - Gets the [pteval_t][pteval_t] associated with the
  specified PTE entry, possibly paravirtualised.

#### Retrieving Page Table Entry Indexes

* [pgd_index()](#pgd_index) - Gets the index of the specified virtual address's
  PGD entry within its PGD.
* [pud_index()](#pud_index) - Gets the index of the specified virtual address's
  PUD entry within its PUD.
* [pmd_index()](#pmd_index) - Gets the index of the specified virtual address's
  PMD entry within its PMD.
* [pte_index()](#pte_index) - Gets the index of the specified virtual address's
  PTE entry within its PTE directory.

#### Retrieving Pages

__NOTE:__ There's room for confusion here - the `pXX_page_vaddr()` functions
each return the virtual address of the physical page referred to by the
specified entry, so e.g. `pgd_page_vaddr()` returns the address of a PUD page,
`pud_page_vaddr()` returns the address of a PMD page, etc.

Equally, the `pXX_page()` functions return a pointer to the [struct page][page]
for the page _referred to_ by the specified entry, e.g. `pgd_page()` returns the
`struct page` for a PUD, `pud_page()` returns the `struct page` for a PMD, etc.

* [pgd_page_vaddr()](#pgd_page_vaddr) - Gets the virtual address of the PUD page
  pointed to by the specified PGD entry.
* [pud_page_vaddr()](#pud_page_vaddr) - Gets the virtual address of the PMD page
  pointed to by the specified PUD entry.
* [pmd_page_vaddr()](#pmd_page_vaddr) - Gets the virtual address of the PTE page
  pointed to by the specified PMD entry.
* [pgd_page()](#pgd_page) - Gets the virtual address of the [struct page][page]
  describing the PUD page pointed to by the specified PGD entry.
* [pud_page()](#pud_page) - Gets the virtual address of the [struct page][page]
  describing the PMD page pointed to by the specified PUD entry.
* [pmd_page()](#pmd_page) - Gets the virtual address of the [struct page][page]
  describing the PTE page pointed to by the specified PMD entry.
* [pte_page()](#pte_page) - Gets the virtual address of the [struct page][page]
  describing the page pointed to by the specified PTE entry.

## Address Translation

### phys_to_virt()

`void *phys_to_virt(phys_addr_t address)`

[phys_to_virt()][phys_to_virt] translates physical address `address` to a
kernel-mapped virtual one.

Wrapper around [__va()][__va].

#### Arguments

* `address` - Physical address to be translated.

#### Returns

Kernel-mapped virtual address.

---

### virt_to_phys()

`phys_addr_t virt_to_phys(volatile void *address)`

[virt_to_phys()][virt_to_phys] translate kernel-mapped virtual address `address`
to a physical one.

Wrapper around [__pa()][__pa].

#### Arguments

* `address` - Kernel-mapped virtual address to be translated.

#### Returns

Physical address associated with specified virtual address.

---

### __va()

`void *__va(phys_addr_t address)`

[__va()][__va] does the heavy lifting for `phys_to_virt()`, see above.

__NOTE:__ Macro, inferring function signature.

---

### __pa()

`phys_addr_t __pa(volatile void *address)`

[__pa()][__pa] does the heavy lifting for `virt_to_phys()`, see above.

__NOTE:__ Macro, inferring function signature.

## Page Tables

### pgd_offset()

`pgd_t *pgd_offset(struct mm_struct *mm, unsigned long address)`

[pgd_offset()][pgd_offset] has a confusing name - it returns a virtual address,
NOT an offset/index. The function locates the PGD _entry_ for the specified
_virtual_ address and returns a _virtual_ address to it.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] associated with the process whose PGD
  we seek.

* `address` - The _virtual_ address whose PGD we seek.

#### Returns

A pointer to (hence virtual address of) a [pgd_t][pgd_t] entry which itself
contains the physical address for the corresponding PUD with associated flags.

---

### pud_offset()

`pud_t *pud_offset(pgd_t *pgd, unsigned long address)`

[pud_offset()][pud_offset] has a confusing name - it returns a virtual address,
NOT an offset/index. The function locates the PUD _entry_ for the specified
_virtual_ address associated with the specified PGD entry, and returns a
_virtual_ address to the PUD entry.

#### Arguments

* `pgd` - A pointer to the PGD entry belonging to the virtual `address`.

* `address` - The _virtual_ address whose PUD we seek.

#### Returns

A pointer to (hence virtual address of) a [pud_t][pud_t] entry which itself
contains the physical address for the corresponding PMD with associated flags.

---

### pmd_offset()

`pmd_t *pmd_offset(pud_t *pud, unsigned long address)`

[pmd_offset()][pmd_offset] has a confusing name - it returns a virtual address,
NOT an offset/index. The function locates the PMD _entry_ for the specified
_virtual_ address associated with the specified PUD entry, and returns a
_virtual_ address to the PMD entry.

#### Arguments

* `pud` - A pointer to the PUD entry belonging to the virtual `address`.

* `address` - The _virtual_ address whose PUD we seek.

#### Returns

A pointer to (hence virtual address of) a [pmd_t][pmd_t] entry which itself
contains the physical address for the corresponding PTE with associated flags.

---

### pte_offset_map()

`pte_t *pte_offset_map(pmd_t *pmd, unsigned long address)`

[pte_offset_map()][pte_offset_map] has a confusing name - it returns a virtual
address, NOT an offset/index. The function locates the PTE _entry_ for the
specified _virtual_ address associated with the specified PMD entry, and returns
a _virtual_ address to the PTE entry.

The `_map` part of the name refers to mapping in the PTE directory if it isn't
already present in [high memory][high-memory]. This is used by architectures
which have a small address space and have [CONFIG_HIGHPTE][CONFIG_HIGHPTE]
set. In x86-64 there is no such pressure on address space so the `_map` is
redundant and the function is therefore essentially a wrapper around
[pte_offset_kernel()][pte_offset_kernel].

__NOTE:__ Macro, inferring function signature.

#### Locking

The appropriate (spin)lock to hold can be obtained via
[pte_lockptr()][pte_lockptr], generally this is acquired _after_ the PTE entry
pointer is obtained. However there is a lot of lockless code in the kernel that
avoids holding this.

The helper function [pte_offset_map_lock()][pte_offset_map_lock] acquires the
PTE lock after retrieving the pointer to the PTE entry. The lock is released via
[pte_unmap_unlock()][pte_unmap_unlock].

#### Arguments

* `pmd` - A pointer to the PMD entry belonging to the virtual `address`.

* `address` - The _virtual_ address whose PTE we seek.

#### Returns

A pointer to (hence virtual address of) a [pte_t][pte_t] entry which itself
contains the physical address for the corresponding physical page with
associated flags.

---

### pte_offset_map_lock()

`pte_t *pte_offset_map_lock(struct mm_struct *mm, pmd_t *pmd, unsigned long address,
                            spinlock_t **ptlp)`

[pte_offset_map_lock()][pte_offset_map_lock] performs the same task as
[pte_offset_map()][pte_offset_map], but also acquires the spinlock returned by
[pte_lockptr()][pte_lockptr] _after_ determining the virtual address of the PTE
entry.

See the above section on `pte_offset_map()` for more details on what the
function is retrieving.

__NOTE:__ Macro, inferring function signature.

#### Locking

The caller is responsible for releasing the acquired spinlock via
[pte_unmap_unlock()][pte_unmap_unlock].

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] associated with the process whose PTE
  we seek.

* `pmd` - A pointer to the PMD entry belonging to the virtual `address`.

* `address` - The _virtual_ address whose PTE we seek.

* `ptlp` - __OUT__ - A pointer to a `spinlock_t *` which will reference the
  spinlock acquired by the function.

#### Returns

A pointer to (hence virtual address of) a [pte_t][pte_t] entry which itself
contains the physical address for the corresponding physical page with
associated flags.

---

### native_pgd_val()

`pgdval_t native_pgd_val(pgd_t pgd)`

[native_pgd_val()][native_pgd_val] extracts the underlying native value of the
specified PGD entry (note the argument is not a pointer.)

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pgdval_t pgd; } pgd_t;`.

On x86-64 [pgdval_t][pgdval_t] is just an `unsigned long`.

#### Arguments

* `pgd` - The [pgd_t][pgd_t] whose value we seek.

#### Returns

The [pgdval_t][pgdval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PGD entry.

---

### native_pud_val()

`pudval_t native_pud_val(pud_t pud)`

[native_pud_val()][native_pud_val] extracts the underlying native value of the
specified PUD entry (note the argument is not a pointer.)

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pudval_t pud; } pud_t;`.

On x86-64 [pudval_t][pudval_t] is just an `unsigned long`.

#### Arguments

* `pud` - The [pud_t][pud_t] whose value we seek.

#### Returns

The [pudval_t][pudval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PUD entry.

---

### native_pmd_val()

`pmdval_t native_pmd_val(pmd_t pmd)`

[native_pmd_val()][native_pmd_val] extracts the underlying native value of the
specified PMD entry (note the argument is not a pointer.)

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pmdval_t pmd; } pmd_t;`.

On x86-64 [pmdval_t][pmdval_t] is just an `unsigned long`.

#### Arguments

* `pmd` - The [pmd_t][pmd_t] whose value we seek.

#### Returns

The [pmdval_t][pmdval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PMD entry.

---

### native_pte_val()

`pteval_t native_pte_val(pte_t pte)`

[native_pte_val()][native_pte_val] extracts the underlying native value of the
specified PTE entry (note the argument is not a pointer.)

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pteval_t pte; } pte_t;`.

On x86-64 [pteval_t][pteval_t] is just an `unsigned long`.

#### Arguments

* `pte` - The [pte_t][pte_t] whose value we seek.

#### Returns

The [pteval_t][pteval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PTE entry.

---

### pgd_val()

`pgdval_t pgd_val(pgd_t pgd)`

[pgd_val()][pgd_val] is a wrapper around [native_pgd_val()][native_pgd_val]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [pgd_val()][pgd_val/para] is used.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pgd` - The [pgd_t][pgd_t] whose value we seek.

#### Returns

The [pgdval_t][pgdval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PGD entry.

---

### pud_val()

`pudval_t pud_val(pud_t pud)`

[pud_val()][pud_val] is a wrapper around [native_pud_val()][native_pud_val]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [pud_val()][pud_val/para] is used.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pud` - The [pud_t][pud_t] whose value we seek.

#### Returns

The [pudval_t][pudval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PUD entry.

---

### pmd_val()

`pmdval_t pmd_val(pmd_t pmd)`

[pmd_val()][pmd_val] is a wrapper around [native_pmd_val()][native_pmd_val]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [pmd_val()][pmd_val/para] is used.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pmd` - The [pmd_t][pmd_t] whose value we seek.

#### Returns

The [pmdval_t][pmdval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PMD entry.

---

### pte_val()

`pteval_t pte_val(pte_t pte)`

[pte_val()][pte_val] is a wrapper around [native_pte_val()][native_pte_val]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [pte_val()][pte_val/para] is used.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pte` - The [pte_t][pte_t] whose value we seek.

#### Returns

The [pteval_t][pteval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PTE entry.

---

### pgd_index()

`unsigned long pgd_index(unsigned long address)`

[pgd_index()][pgd_index] returns the index of the specified _virtual_ address's
associated PGD entry within its PGD page.

Directly from the code:

```c
/*
 * the pgd page can be thought of an array like this: pgd_t[PTRS_PER_PGD]
 *
 * this macro returns the index of the entry in the pgd page which would
 * control the given virtual address
*/
```

On x86-64, [PTRS_PER_PGD][PTRS_PER_PGD] is 512.

#### Arguments

* `address` - _Virtual_ address whose PGD entry index we seek.

#### Returns

Index of the virtual address's PGD entry within its PGD.

---

### pud_index()

`unsigned long pud_index(unsigned long address)`

[pud_index()][pud_index] returns the index of the specified _virtual_ address's
associated PUD entry within its PUD page.

Adapted from the code comment for [pgd_index()][pgd_index]:

The PUD page can be thought of an array like this: `pud_t[PTRS_PER_PUD]`. This
macro returns the index of the entry in the pud page which would control the
given virtual address.

On x86-64, [PTRS_PER_PUD][PTRS_PER_PUD] is 512.

#### Arguments

* `address` - _Virtual_ address whose PUD entry index we seek.

#### Returns

Index of the virtual address's PUD entry within its PUD.

---

### pmd_index()

`unsigned long pmd_index(unsigned long address)`

[pmd_index()][pmd_index] returns the index of the specified _virtual_ address's
associated PMD entry within its PMD page.

Adapted from the code comment for [pgd_index()][pgd_index]:

The PMD page can be thought of an array like this: `pmd_t[PTRS_PER_PMD]`. This
macro returns the index of the entry in the pmd page which would control the
given virtual address.

On x86-64, [PTRS_PER_PMD][PTRS_PER_PMD] is 512.

#### Arguments

* `address` - _Virtual_ address whose PMD entry index we seek.

#### Returns

Index of the virtual address's PMD entry within its PMD.

---

### pte_index()

`unsigned long pte_index(unsigned long address)`

[pte_index()][pte_index] returns the index of the specified _virtual_ address's
associated PTE entry within its PTE page.

Adapted from the code comment for [pgd_index()][pgd_index]:

The PTE page can be thought of an array like this: `pte_t[PTRS_PER_PTE]`. This
macro returns the index of the entry in the pte page which would control the
given virtual address.

On x86-64, [PTRS_PER_PTE][PTRS_PER_PTE] is 512.

#### Arguments

* `address` - _Virtual_ address whose PTE entry index we seek.

#### Returns

Index of the virtual address's PTE entry within its PTE directory.

---

### pgd_page_vaddr()

`unsigned long pgd_page_vaddr(pgd_t pgd)`

[pgd_page_vaddr()][pgd_page_vaddr] returns the _virtual_ address of the PUD page
stored in the specified `pgd` PGD _entry_. As a result, and rather confusingly,
`pgd_page_vaddr()` returns the virtual address of a PUD page.

This can be seen as essentially dereferencing the PGD - though the function has
the added steps of masking out any flags and converting the physical address to
a virtual one.

#### Arguments

* `pgd` - PGD entry whose pointed-at PUD page is desired.

#### Returns

The _virtual_ address of the PUD page.

---

### pud_page_vaddr()

`unsigned long pud_page_vaddr(pud_t pud)`

[pud_page_vaddr()][pud_page_vaddr] returns the _virtual_ address of the PMD page
stored in the specified `pud` PUD _entry_. As a result, and rather confusingly,
`pud_page_vaddr()` returns the virtual address of a PMD page.

This can be seen as essentially dereferencing the PUD - though the function has
the added steps of masking out any flags and converting the physical address to
a virtual one.

#### Arguments

* `pud` - PUD entry whose pointed-at PMD page is desired.

#### Returns

The _virtual_ address of the PMD page.

---

### pmd_page_vaddr()

`unsigned long pmd_page_vaddr(pmd_t pmd)`

[pmd_page_vaddr()][pmd_page_vaddr] returns the _virtual_ address of the PTE page
stored in the specified `pmd` PMD _entry_. As a result, and rather confusingly,
`pmd_page_vaddr()` returns the virtual address of a PTE page.

This can be seen as essentially dereferencing the PMD - though the function has
the added steps of masking out any flags and converting the physical address to
a virtual one.

#### Arguments

* `pmd` - PMD entry whose pointed-at PTE page is desired.

#### Returns

The _virtual_ address of the PTE page.

---

### pgd_page()

`struct page *pgd_page(pgd_t pgd)`

[pgd_page()][pgd_page] returns the virtual address of the [struct page][page] of
the PUD pointed to by the specified PGD entry. This is a bit confusing, as
`pgd_page()` returns the [struct page][page] for a PUD, not a PGD.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pgd` - PGD entry whose pointed-at PUD page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PGD entry's
pointed-at PUD page.

---

### pud_page()

`struct page *pud_page(pud_t pud)`

[pud_page()][pud_page] returns the virtual address of the [struct page][page] of
the PMD pointed to by the specified PUD entry. This is a bit confusing, as
`pud_page()` returns the [struct page][page] for a PMD, not a PUD.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pud` - PUD entry whose pointed-at PMD page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PUD entry's
pointed-at PMD page.

---

### pmd_page()

`struct page *pmd_page(pmd_t pmd)`

[pmd_page()][pmd_page] returns the virtual address of the [struct page][page] of
the PTE pointed to by the specified PMD entry. This is a bit confusing, as
`pmd_page()` returns the [struct page][page] for a PTE, not a PMD.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pmd` - PMD entry whose pointed-at PTE page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PMD entry's
pointed-at PTE page.

---

### pte_page()

`struct page *pte_page(pte_t pte)`

[pte_page()][pte_page] returns the virtual address of the [struct page][page] of
the physical page pointed to by the specified PTE entry. This is a bit
confusing, as `pte_page()` returns the [struct page][page] for the ultimate
physical page, not a PTE.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pte` - PTE entry whose pointed-at physical page's [struct page][page] is
  desired.

#### Returns

The virtual address of the [struct page][page] describing the PTE entry's
pointed-at physical page.

---

[linux-4.6]:https://github.com/torvalds/linux/tree/v4.6/

[pgdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44

[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40

[pgd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L714
[mm_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pud_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L684
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pmd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L639
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pte_offset_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L140
[high-memory]:https://en.wikipedia.org/wiki/High_memory
[CONFIG_HIGHPTE]:http://cateee.net/lkddb/web-lkddb/HIGHPTE.html
[pte_offset_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L600
[pte_lockptr]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1619
[pte_offset_map_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1680
[pte_unmap_unlock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1689
[pte_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[native_pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L259
[native_pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L277
[native_pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L298
[native_pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L352
[pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L73
[paravirtualisation]:https://en.wikipedia.org/wiki/Paravirtualization
[pgd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L421
[pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L77
[pud_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L554
[pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L82
[pmd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L514
[pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L86
[pte_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L393
[pgd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L708
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[pud_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L679
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[pmd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L575
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[pte_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L595
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[pgd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L667
[pud_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L626
[pmd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L557
[pgd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L676
[pud_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L635
[pmd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L566
[pte_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L171
