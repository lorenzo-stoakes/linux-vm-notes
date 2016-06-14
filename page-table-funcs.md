# Page Table Functions

## Contents

### Retrieving Page Table Entries

__NOTE:__ Confusingly, the `_offset` functions actually return a virtual address
for the page table entry associated with the specified virtual address.

* [pgd_offset()](#pgd_offset) - Gets the virtual address of the specified
  virtual address's entry in the specified [struct mm_struct][mm_struct]'s PGD.
* [pgd_offset_k()](#pgd_offset_k) - Gets the virtual address of the specified
  virtual address's entry in the kernel's PGD.
* [pud_offset()](#pud_offset) - Gets the virtual address of specified virtual
  address's PUD entry.
* [pmd_offset()](#pmd_offset) - Gets the virtual address of specified virtual
  address's PMD entry.
* [pte_offset_map()](#pte_offset_map) - Gets the virtual address of the
  specified virtual address's PTE entry (always mapped in x86-64.)
* [pte_offset_map_lock()](#pte_offset_map_lock) - Gets the virtual address of
  the specified virtual address's PTE entry then acquires a PTE lock.

### Retrieving Page Table Entry Values

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
* [pgprot_val()](#pgprot_val) - Gets the [pgprotval_t][pgprotval_t] associated
  with the specified [pgprot_t][pgprot_t] flag bitfield.
* [pud_pfn()](#pud_pfn) - Gets the Page Frame Number (PFN) of the PMD physical
  page contained in the specified PUD entry.
* [pmd_pfn()](#pmd_pfn) - Gets the Page Frame Number (PFN) of the PTE physical
  page contained in the specified PMD entry.
* [pte_pfn()](#pte_pfn) - Gets the Page Frame Number (PFN) of the physical page
  contained in the specified PTE entry.

### Masking Page Table Entry Values

__NOTE:__ There are no `pgd` or `pte` versions of these functions, as the masks
for a PGD or PTE are simply `PTE_PFN_MASK` and `PTE_FLAGS_MASK` for the PFN and
flags masks respectively.

* [pud_pfn_mask()](#pud_pfn_mask) - Returns a [bitmask][bitmask] for obtaining
  the PMD physical address from a PUD entry.
* [pmd_pfn_mask()](#pmd_pfn_mask) - Returns a [bitmask][bitmask] for obtaining
  the PTE physical address from a PMD entry.
* [pud_flags_mask()](#pud_flags_mask) - Returns a [bitmask][bitmask] for
  obtaining a PUD entry's flags.
* [pmd_flags_mask()](#pmd_flags_mask) - Returns a [bitmask][bitmask] for
  obtaining a PMD entry's flags.

### Creating Page Table Entries

* [native_make_pgd()](#native_make_pgd) - Converts the specified
  [pgdval_t][pgdval_t] into a [pgd_t][pgd_t].
* [native_make_pud()](#native_make_pud) - Converts the specified
  [pudval_t][pudval_t] into a [pud_t][pud_t].
* [native_make_pmd()](#native_make_pmd) - Converts the specified
  [pmdval_t][pmdval_t] into a [pmd_t][pmd_t].
* [native_make_pte()](#native_make_pte) - Converts the specified
  [pteval_t][pteval_t] into a [pte_t][pte_t].
* [__pgd()](#__pgd) - Converts the specified [pgdval_t][pgdval_t] into a
  [pgd_t][pgd_t], possibly paravirtualised.
* [__pud()](#__pud) - Converts the specified [pudval_t][pudval_t] into a
  [pud_t][pud_t], possibly paravirtualised.
* [__pmd()](#__pmd) - Converts the specified [pmdval_t][pmdval_t] into a
  [pmd_t][pmd_t], possibly paravirtualised.
* [__pte()](#__pte) - Converts the specified [pteval_t][pteval_t] into a
  [pte_t][pte_t], possibly paravirtualised.
* [pfn_pmd()](#pfn_pmd) - Creates a PMD entry pointing at the specified PFN with
  the specified flags.
* [pfn_pte()](#pfn_pte) - Creates a PTE entry pointing at the specified PFN with
  the specified flags.

### Allocating/Freeing Page Table Entries

* [pgd_alloc()](#pgd_alloc) - Allocates a new PGD page for the specified
  [struct mm_struct][mm_struct].
* [pgd_free()](#pgd_free) - Frees the specified [struct mm_struct][mm_struct]'s
  specified PGD page.
* [pud_alloc()](#pud_alloc) - If necessary, allocates a new PUD page for the
  specified address and returns pointer to address's PUD entry.
* [pud_free()](#pud_free) - Frees the specified PUD page.

### Retrieving Page Table Entry Indexes

* [pgd_index()](#pgd_index) - Gets the index of the specified virtual address's
  PGD entry within its PGD.
* [pud_index()](#pud_index) - Gets the index of the specified virtual address's
  PUD entry within its PUD.
* [pmd_index()](#pmd_index) - Gets the index of the specified virtual address's
  PMD entry within its PMD.
* [pte_index()](#pte_index) - Gets the index of the specified virtual address's
  PTE entry within its PTE directory.

### Retrieving Pages

__NOTE:__ Confusingly, `pXX_page[_vaddr]()` deals with the pointed at entry, so
e.g. `pgd_page[_vaddr]()` return a PUD `struct page`/virtual address, etc.

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

### Page Table Entry State

* [pgd_none()](#pgd_none) - Determines if the specified PGD entry is empty.
* [pud_none()](#pud_none) - Determines if the specified PUD entry is empty.
* [pmd_none()](#pmd_none) - Determines if the specified PMD entry is empty.
* [pte_none()](#pte_none) - Determines if the specified PTE entry is empty.
* [pgd_bad()](#pgd_bad) - Determines if the specified PGD entry or its
  descendants are not in a safe state to be modified.
* [pud_bad()](#pud_bad) - Determines if the specified PUD entry or its
  descendants are not in a safe state to be modified.
* [pmd_bad()](#pmd_bad) - Determines if the specified PMD entry or its
  descendants are not in a safe state to be modified.

### Retrieving Flag Bitfields

* [pgd_flags()](#pgd_flags) - Retrieves bitfield containing the specified PGD
  entry's flags.
* [pud_flags()](#pud_flags) - Retrieves bitfield containing the specified PUD
  entry's flags.
* [pmd_flags()](#pmd_flags) - Retrieves bitfield containing the specified PMD
  entry's flags.
* [pte_flags()](#pte_flags) - Retrieves bitfield containing the specified PTE
  entry's flags.
* [pud_pgprot()](#pud_pgprot) - Retrieves bitfield containing the specified PUD
  entry's flags wrapped in a [pgprot_t][pgprot_t].
* [pmd_pgprot()](#pmd_pgprot) - Retrieves bitfield containing the specified PMD
  entry's flags wrapped in a [pgprot_t][pgprot_t].
* [pte_pgprot()](#pte_pgprot) - Retrieves bitfield containing the specified PTE
  entry's flags wrapped in a [pgprot_t][pgprot_t].

### Creating Flag Bitfields

* [__pgprot()](#__pgprot) - Converts the specified [pgprotval_t][pgprotval_t]
  into a [pgprot_t][pgprot_t].
* [massage_pgprot()](#massage_pgprot) - Masks the specified [pgprot_t][pgprot_t]
  fields against all possible flags, returns [pgprotval_t][pgprotval_t].
* [canon_pgprot()](#canon_pgprot) -  Masks the specified [pgprot_t][pgprot_t]
  fields against all possible flags, returns [pgprot_t][pgprot_t].

### Setting/Clearing Flag Bitfields

* [pmd_set_flags()](#pmd_set_flags) - Returns a new copy of a PMD with the
  specified bitfield appended to its flag bitfield.
* [pte_set_flags()](#pte_set_flags) - Returns a new copy of a PTE with the
  specified bitfield appended to its flag bitfield.
* [pmd_clear_flags()](#pmd_clear_flags) - Returns a new copy of a PMD with the
  specified bitfield cleared from its flag bitfield.
* [pte_clear_flags()](#pte_clear_flags) - Returns a new copy of a PTE with the
  specified bitfield cleared from its flag bitfield.

### Retrieving Individual Flags

__NOTE:__ Confusingly, the `pXX_<flag>()` functions retrieve flags from the
specified `pXX` entry, however they refer to the pointed at page,
e.g. `pgd_present()` determines if the pointed at PUD page is present.

* [pgd_present()](#pgd_present) - Determines if the pointed at PUD page is
  present, i.e. resident in memory.
* [pud_present()](#pud_present) - Determines if the pointed at PMD page is
  present, i.e. resident in memory.
* [pmd_present()](#pmd_present) - Determines if the pointed at PTE page is
  present, i.e. resident in memory.
* [pte_present()](#pte_present) - Determines if the pointed at physical page is
  present, i.e. resident in memory.
* [pmd_young()](#pmd_young) - Determines if the PTE page pointed at by the
  specified PMD entry has been accessed.
* [pte_young()](#pte_young) - Determines if the physical page pointed at by the
  specified PTE entry has been accessed.
* [pmd_dirty()](#pmd_dirty) - Determines if the PTE page pointed at by the
  specified PMD entry has been modified.
* [pte_dirty()](#pte_dirty) - Determines if the physical page pointed at by the
  specified PTE entry has been modified.
* [pmd_write()](#pmd_write) - Determines if the PTE page pointed at by the
  specified PMD entry is writable.
* [pte_write()](#pte_write) - Determines if the physical page pointed at by the
  specified PTE entry is writable.
* [pte_exec()](#pte_exec) - Determines if the physical page pointed at by the
  specified PTE entry is executable.
* [pte_global()](#pte_global) - Determines whether ordinary [TLB][tlb] flushes
  do not clear the specified PTE entry mapping.
* [pte_special()](#pte_special) - Determines if the PTE entry is marked special.
* [pmd_soft_dirty()](#pmd_soft_dirty) - Determines if the PMD entry is marked
  [soft-dirty][soft-dirty].
* [pte_soft_dirty()](#pte_soft_dirty) - Determines if the PTE entry is marked
  [soft-dirty][soft-dirty].
* [pmd_devmap()](#pmd_devmap) - Determines if the PTE page pointed at by the
  specified PMD entry is part of a [device mapping][device-mapper].
* [pte_devmap()](#pte_devmap) - Determines if the physical page pointed at by
  the specified PTE entry is part of a [device mapping][device-mapper].

#### Huge Pages

* [pud_huge()](#pud_huge) - Determines if the PMD page pointed at by the
  specified PUD entry is huge in the context of [hugetlb][hugetlb].
* [pmd_huge()](#pmd_huge) - Determines if the PTE page pointed at by the
  specified PMD entry is huge in the context of [hugetlb][hugetlb].
* [pte_huge()](#pte_huge) - Determines if the physical page pointed at by the
  specified PTE entry is huge (without context.)
* [pmd_trans_huge()](#pmd_trans_huge) - Determines if the PTE page pointed at by
  the specified PMD entry is a [transparent huge page][transhuge].
* [pgd_large()](#pgd_large) - Determines if the pointed at PUD page is huge
  (without context.)
* [pud_large()](#pud_large) - Determines if the pointed at PMD page is huge
  (without context.)
* [pmd_large()](#pmd_large) - Determines if the pointed at PTE page is huge
  (without context.)

### Setting/Clearing Individual Flags

#### Accessed Flag

* [pmd_mkyoung()](#pmd_mkyoung) - Returns a new copy of the specified PMD entry
  with the accessed flag set.
* [pmd_mkold()](#pmd_mkold) - Returns a new copy of the specified PMD entry with
  the accessed flag cleared.
* [pte_mkyoung()](#pte_mkyoung) - Returns a new copy of the specified PTE entry
  with the accessed flag set.
* [pte_mkold()](#pte_mkold) - Returns a new copy of the specified PTE entry with
  the accessed flag cleared.

#### Dirty Flag

* [pmd_mkdirty()](#pmd_mkdirty) - Returns a new copy of the specified PMD entry
  with the dirty (modified) flag set.
* [pmd_mkclean()](#pmd_mkclean) - Returns a new copy of the specified PMD entry
  with the dirty (modified) flag cleared.
* [pte_mkdirty()](#pte_mkdirty) - Returns a new copy of the specified PTE entry
  with the dirty (modified) flag set.
* [pte_mkclean()](#pte_mkclean) - Returns a new copy of the specified PTE entry
  with the dirty (modified) flag cleared.

#### Read/Write Flag

* [pmd_mkwrite()](#pmd_mkwrite) - Returns a new copy of the specified PMD entry
  with the read/write flag set.
* [pmd_wrprotect()](#pmd_wrprotect) - Returns a new copy of the specified PMD
  entry with the read/write flag cleared.
* [pte_mkwrite()](#pte_mkwrite) - Returns a new copy of the specified PTE entry
  with the read/write flag set.
* [pte_wrprotect()](#pte_wrprotect) - Returns a new copy of the specified PTE
  entry with the read/write flag cleared.

#### Soft-Dirty Flag

* [pmd_mksoft_dirty()](#pmd_mksoft_dirty) - Returns a new copy of the specified
  PMD entry with the [soft-dirty][soft-dirty] flag set.
* [pmd_clear_soft_dirty()](#pmd_clear_soft_dirty) - Returns a new copy of the
  specified PMD entry with the [soft-dirty][soft-dirty] flag cleared.
* [pte_mksoft_dirty()](#pte_mksoft_dirty) - Returns a new copy of the specified
  PTE entry with the [soft-dirty][soft-dirty] flag set.
* [pte_clear_soft_dirty()](#pte_clear_soft_dirty) - Returns a new copy of the
  specified PTE entry with the [soft-dirty][soft-dirty] flag cleared.

#### dev-map Flag

* [pmd_mkdevmap()](#pmd_mkdevmap) - Returns a new copy of the specified PMD
  entry with the [devmap][device-mapper] flag set.
* [pte_mkdevmap()](#pte_mkdevmap) - Returns a new copy of the specified PTE
  entry with the [devmap][device-mapper] flag cleared.

#### Present Flag

* [pmd_mknotpresent()](#pmd_mknotpresent) - Returns a new copy of the specified
  PMD entry with the present flag cleared, i.e. indicating the underlying PTE
  page is not resident.

#### Global Flag

* [pte_mkglobal()](#pte_mkglobal) - Returns a new copy of the specified PTE
  entry with the global flag set, avoiding [TLB][tlb] flushes.
* [pte_clrglobal()](#pte_clrglobal) - Returns a new copy of the specified PTE
  entry with the global flag cleared.

#### No-Execute (NX) flag

* [pte_mkexec()](#pte_mkexec) - Returns a new copy of the specified PTE entry
  with the NX flag cleared, marking the underlying physical page executable.

#### Special Flag

* [pte_mkspecial()](#pte_mkspecial) - Returns a new copy of the specified PTE
  entry with the special flag set.

#### Huge Pages

* [pmd_mkhuge()](#pmd_mkhuge) - Returns a new copy of the specified PMD entry
  with the huge page flag set.
* [pte_mkhuge()](#pte_mkhuge) - Returns a new copy of the specified PTE entry
  with the huge page flag set.
* [pte_clrhuge()](#pte_clrhuge) - Returns a new copy of the specified PTE entry
  with the huge page flag cleared.

---

### pgd_offset()

`pgd_t *pgd_offset(struct mm_struct *mm, unsigned long address)`

[pgd_offset()][pgd_offset] has a confusing name - it returns a virtual address,
NOT an offset/index. The function locates the _entry_ for the specified
_virtual_ address in the PGD taken from the specified
[struct mm_struct][mm_struct] and returns a _virtual_ address to it.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] associated with the process whose PGD
  we seek.

* `address` - The _virtual_ address whose PGD entry we seek.

#### Returns

A pointer to (hence virtual address of) a [pgd_t][pgd_t] entry which itself
contains the physical address for the corresponding PUD with associated flags.

---

### pgd_offset_k()

`pgd_t *pgd_offset_k(unsigned long address)`

[pgd_offset_k()][pgd_offset_k] has a confusing name - it returns a virtual
address, NOT an offset/index. The function locates the _entry_ for the specified
_virtual_ address inside the kernel's PGD, and returns a _virtual_ address to
it.

The macro is simply defined as:

```c
#define pgd_offset_k(address) pgd_offset(&init_mm, (address))
```

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `address` - The _virtual_ address whose PGD entry we seek.

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

* `address` - The _virtual_ address whose PUD entry we seek.

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

* `address` - The _virtual_ address whose PMD entry we seek.

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
specified PGD entry which itself is either empty, contains the physical address
of a PUD page and associated flags, or contains swap metadata with associated
flags and the 'present' flag unset.

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
specified PUD entry which itself is either empty, contains the physical address
of a PMD page and associated flags, or contains swap metadata with associated
flags and the 'present' flag unset.

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
specified PTE entry which itself is either empty, contains the physical address
of a PUD page and associated flags, or contains swap metadata with associated
flags and the 'present' flag unset.

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
specified PTE entry which itself is either empty, contains the physical address
of a physical page and associated flags, or contains swap metadata with
associated flags and the 'present' flag unset.

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

See `native_pgd_val()` entry above for more details on returned value.

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

See `native_pud_val()` entry above for more details on returned value.

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

See `native_pmd_val()` entry above for more details on returned value.

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

See `native_pte_val()` entry above for more details on returned value.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pte` - The [pte_t][pte_t] whose value we seek.

#### Returns

The [pteval_t][pteval_t] (i.e. `unsigned long` on x86-64) associated with the
specified PTE entry.

---

### pgprot_val()

`pgprotval_t pgprot_val(pgprot_t val)`

[pgprot_val()][pgprot_val] retrieves the native [pgprotval_t][pgprotval_t]
(`unsigned long` in x86-64) value associated with the specified
[pgprot_t][pgprot_t] protection flags bit field.

As with `pXX_t` the reason this has to exist is that `pgprot_t` is a `typedef
struct` used to enforce type safety - `typedef struct { pgprotval_t pgprot; }
pgprot_t;`.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `val` - The page protection flags whose [pgprotval_t][pgprotval_t] native
  value we desire.

#### Returns

The [pgprotval_t][pgprotval_t] containing the native bitfield.

---

### pud_pfn()

`unsigned long pud_pfn(pud_t pud)`

[pud_pfn()][pud_pfn] determines the Page Frame Number (PFN) of the PMD page's
physical address contained within the specified PUD entry.

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

Obviously, if the entry is empty or not present the returned PFN will be
invalid.

If you think of memory as an array of physical pages, the PFN is simply the
index of the page within that array.

#### Arguments

* `pud` - The [pud_t][pud_t] which contains the PMD page's physical address
  whose PFN we desire.

#### Returns

The PFN of the PMD page pointed at by the specified PUD entry.

---

### pmd_pfn()

`unsigned long pmd_pfn(pmd_t pmd)`

[pmd_pfn()][pmd_pfn] determines the Page Frame Number (PFN) of the PTE page's
physical address contained within the specified PMD entry.

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

Obviously, if the entry is empty or not present the returned PFN will be
invalid.

If you think of memory as an array of physical pages, the PFN is simply the
index of the page within that array.

#### Arguments

* `pmd` - The [pmd_t][pmd_t] which contains the PTE page's physical address
  whose PFN we desire.

#### Returns

The PFN of the PTE page pointed at by the specified PMD entry.

---

### pte_pfn()

`unsigned long pte_pfn(pte_t pte)`

[pte_pfn()][pte_pfn] determines the Page Frame Number (PFN) of the physical
address contained within the specified PTE entry.

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

Obviously, if the entry is empty or not present the returned PFN will be
invalid.

If you think of memory as an array of physical pages, the PFN is simply the
index of the page within that array.

#### Arguments

* `pte` - The [pte_t][pte_t] which contains the physical address whose PFN we
  desire.

#### Returns

The PFN of the physical address contained in the PTE entry.

---

### pud_pfn_mask()

`pudval_t pud_pfn_mask(pud_t pud)`

[pud_pfn_mask()][pud_pfn_mask] returns a [bitmask][bitmask] for a PUD entry to
obtain the PMD page's physical address it points at.

Note that neither PGD or PTE entries require a bitmask function, rather they use
`PTE_PFN_MASK` and `PTE_FLAGS_MASK`. PUD and PMD entries need special treatment
in case they refer to huge/gigantic pages.

If the PUD entry indicates it refers to a gigantic page (i.e. `_PAGE_PSE` is
set), `PHYSICAL_PUD_PAGE_MASK` is returned, otherwise `PTE_PFN_MASK` is
returned. `PHYSICAL_PUD_PAGE_MASK` masks out the lower `PUD_SHIFT` bits, i.e. 30
bits on x86-64, meaning the PUD page table entry refers to a 1GiB physical page.

#### Arguments

* `pud` - The PUD entry whose PMD page physical address mask we desire.

#### Returns

A mask for a PUD entry to obtain the physical address of the PMD page it points
at.

---

### pmd_pfn_mask()

`pmdval_t pmd_pfn_mask(pmd_t pmd)`

[pmd_pfn_mask()][pmd_pfn_mask] returns a [bitmask][bitmask] for a PMD entry to
obtain the PTE page's physical address it points at.

Note that neither PGD or PTE entries require a bitmask function, rather they use
`PTE_PFN_MASK` and `PTE_FLAGS_MASK`. PMD and PMD entries need special treatment
in case they refer to huge/gigantic pages.

If the PMD entry indicates it refers to a huge page (i.e. `_PAGE_PSE` is set),
`PHYSICAL_PMD_PAGE_MASK` is returned, otherwise `PTE_PFN_MASK` is
returned. `PHYSICAL_PMD_PAGE_MASK` masks out the lower `PMD_SHIFT` bits, i.e. 21
bits on x86-64, meaning the PMD page table entry refers to a 2MiB physical page.

#### Arguments

* `pmd` - The PMD entry whose PTE page physical address mask we desire.

#### Returns

A mask for a PMD entry to obtain the physical address of the PTE page it points
at.

---

### pud_flags_mask()

`pudval_t pud_flags_mask(pud_t pud)`

[pud_flags_mask()][pud_flags_mask] returns a [bitmask][bitmask] for a PUD entry
to obtain its flags. It is simply the bitwise complement of
[pud_pfn_mask()][pud_pfn_mask], see the entry above for this for more details.

#### Arguments

* `pud` - The PUD entry whose flags we desire.

#### Returns

A mask for a PUD entry to obtain its flags.

---

### pmd_flags_mask()

`pmdval_t pmd_flags_mask(pmd_t pmd)`

[pmd_flags_mask()][pmd_flags_mask] returns a [bitmask][bitmask] for a PMD entry
to obtain its flags. It is simply the bitwise complement of
[pmd_pfn_mask()][pmd_pfn_mask], see the entry above for this for more details.

#### Arguments

* `pmd` - The PMD entry whose flags we desire.

#### Returns

A mask for a PMD entry to obtain its flags.

---

### native_make_pgd()

`pgd_t native_make_pgd(pgdval_t val)`

[native_make_pgd()][native_make_pgd] converts the specified [pgdval_t][pgdval_t]
into a [pgd_t][pgd_t].

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pgdval_t pgd; } pgd_t;`.

On x86-64 [pgdval_t][pgdval_t] is just an `unsigned long`.

#### Arguments

* `val` - The [pgdval_t][pgdval_t] which is to be wrapped into a [pgd_t][pgd_t].

#### Returns

A [pgd_t][pgd_t] containing the specified [pgdval_t][pgdval_t].

---

### native_make_pud()

`pud_t native_make_pud(pudval_t val)`

[native_make_pud()][native_make_pud] converts the specified [pudval_t][pudval_t]
into a [pud_t][pud_t].

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pudval_t pud; } pud_t;`.

On x86-64 [pudval_t][pudval_t] is just an `unsigned long`.

#### Arguments

* `val` - The [pudval_t][pudval_t] which is to be wrapped into a [pud_t][pud_t].

#### Returns

A [pud_t][pud_t] containing the specified [pudval_t][pudval_t].

---

### native_make_pmd()

`pmd_t native_make_pmd(pmdval_t val)`

[native_make_pmd()][native_make_pmd] converts the specified [pmdval_t][pmdval_t]
into a [pmd_t][pmd_t].

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pmdval_t pmd; } pmd_t;`.

On x86-64 [pmdval_t][pmdval_t] is just an `unsigned long`.

#### Arguments

* `val` - The [pmdval_t][pmdval_t] which is to be wrapped into a [pmd_t][pmd_t].

#### Returns

A [pmd_t][pmd_t] containing the specified [pmdval_t][pmdval_t].

---

### native_make_pte()

`pte_t native_make_pte(pteval_t val)`

[native_make_pte()][native_make_pte] converts the specified [pteval_t][pteval_t]
into a [pte_t][pte_t].

The reason this has to exist is that each `pXX_t` is a `typedef struct` used to
enforce type safety - `typedef struct { pteval_t pte; } pte_t;`.

On x86-64 [pteval_t][pteval_t] is just an `unsigned long`.

#### Arguments

* `val` - The [pteval_t][pteval_t] which is to be wrapped into a [pte_t][pte_t].

#### Returns

A [pte_t][pte_t] containing the specified [pteval_t][pteval_t].

---

### __pgd()

`pgd_t __pgd(pgdval_t val)`

[__pgd()][__pgd] is a wrapper around [native_make_pgd()][native_make_pgd]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [__pgd()][__pgd/para] is used.

See `native_make_pgd()` entry above for more details on returned value.

#### Arguments

* `val` - The [pgdval_t][pgdval_t] which is to be wrapped into a [pgd_t][pgd_t].

#### Returns

A [pgd_t][pgd_t] containing the specified [pgdval_t][pgdval_t].

---

### __pud()

`pud_t __pud(pudval_t val)`

[__pud()][__pud] is a wrapper around [native_make_pud()][native_make_pud]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [__pud()][__pud/para] is used.

See `native_make_pud()` entry above for more details on returned value.

#### Arguments

* `val` - The [pudval_t][pudval_t] which is to be wrapped into a [pud_t][pud_t].

#### Returns

A [pud_t][pud_t] containing the specified [pudval_t][pudval_t].

---

### __pmd()

`pmd_t __pmd(pmdval_t val)`

[__pmd()][__pmd] is a wrapper around [native_make_pmd()][native_make_pmd]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [__pmd()][__pmd/para] is used.

See `native_make_pmd()` entry above for more details on returned value.

#### Arguments

* `val` - The [pmdval_t][pmdval_t] which is to be wrapped into a [pmd_t][pmd_t].

#### Returns

A [pmd_t][pmd_t] containing the specified [pmdval_t][pmdval_t].

---

### __pte()

`pte_t __pte(pteval_t val)`

[__pte()][__pte] is a wrapper around [native_make_pte()][native_make_pte]
unless the system is [paravirtualised][paravirtualisation] in which case a
different [__pte()][__pte/para] is used.

See `native_make_pte()` entry above for more details on returned value.

#### Arguments

* `val` - The [pteval_t][pteval_t] which is to be wrapped into a [pte_t][pte_t].

#### Returns

A [pte_t][pte_t] containing the specified [pteval_t][pteval_t].

---

### pfn_pmd()

`pmd_t pfn_pmd(unsigned long page_nr, pgprot_t pgprot)`

[pfn_pmd()][pfn_pmd] takes the specified `page_nr` Page Frame Number (PFN) of
the PTE this PMD will point at and the page flags `pgprot` and bitwise combines
them together before wrapping in a [pmd_t][pmd_t] via [__pmd()][__pmd].

The function uses [massage_pgprot()][massage_pgprot] to mask the `pgprot` value
against all possible flag to avoid setting the flag bitfield to something
invalid.

#### Arguments

* `page_nr` - The PFN of the PTE page which we want the PMD entry to point at.

* `pgprot` - The flags we want the PMD entry to contain.

#### Returns

A [pmd_t][pmd_t] PMD entry pointing at the specified PTE page with the specified
flags.

---

### pfn_pte()

`pte_t pfn_pte(unsigned long page_nr, pgprot_t pgprot)`

[pfn_pte()][pfn_pte] takes the specified `page_nr` Page Frame Number (PFN) of
the physical page this PTE will point at and the page flags `pgprot` and bitwise
combines them together before wrapping in a [pte_t][pte_t] via [__pte()][__pte].

The function uses [massage_pgprot()][massage_pgprot] to mask the `pgprot` value
against all possible flag to avoid setting the flag bitfield to something
invalid.

#### Arguments

* `page_nr` - The PFN of the physical page which we want the PTE entry to point
  at.

* `pgprot` - The flags we want the PTE entry to contain.

#### Returns

A [pte_t][pte_t] PTE entry pointing at the specified physical page with the
specified flags.

---

### pgd_alloc()

`pgd_t *pgd_alloc(struct mm_struct *mm)`

[pgd_alloc()][pgd_alloc] allocates a single page for a PGD page via
[_pgd_alloc()][_pgd_alloc] and [__get_free_page()][__get_free_page]. For x86-64
systems it is as simple as this, for other x86 variants there is additional
complexity.

The allocation uses the [PGALLOC_GFP][PGALLOC_GFP] 'Get Free Page' (GFP)
bitfield which uses the usual kernel `GFP_KERNEL` bitset, as well as:

* `__GFP_NOTRACK` - Avoid tracking with kmemcheck.
* `__GFP_REPEAT` - Try hard to allocate the memory.
* `__GFP_ZERO` - Zero the page.

Once the page is allocated, [pgd_ctor()][pgd_ctor] is called to perform a few
tasks:

1. kernel mappings are copied into the PGD via
   [clone_pgd_range()][clone_pgd_range].
2. The underlying [struct page][page] associated with the PGD has its `index`
   field set to the [struct mm_struct][mm_struct]'s virtual address via
   [pgd_set_mm()][pgd_set_mm].
3. The [struct page][page] is added to the `pgd_list` list via
   [pgd_list_add()][pgd_list_add].

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] whose PGD needs populating.

#### Returns

A pointer to the physical page containing an array of `PTRS_PER_PGD`
[pgd_t][pgd_t]s.

---

### pgd_free()

`void pgd_free(struct mm_struct *mm, pgd_t *pgd)`

[pgd_free()][pgd_free] frees the specified PGD page from the specified
[struct mm_struct][mm_struct].

It starts by calling [pgd_dtor()][pgd_dtor] which removes the underlying
[struct page][page] from the `pgd_list` via [pgd_list_del()][pgd_list_del].

[_pgd_free()][_pgd_free] then performs the actual freeing of the PGD page via
[free_page()][free_page].

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] whose PGD needs freeing.

* `pgd` - The PGD page which is to be freed.

#### Returns

N/A

---

### pud_alloc()

`pud_t *pud_alloc(struct mm_struct *mm, pgd_t *pgd, unsigned long address)`

[pud_alloc()][pud_alloc] first checks to see whether the specified PGD entry
`pgd` is empty, if so it allocates a new PUD page and modifies the PGD entry to
point at that page, otherwise it leaves the existing mapping in place. It then
returns the PUD entry indexed by the virtual `address` within that page.

If allocation is required, this is performed via [__pud_alloc()][__pud_alloc]
which subsequently calls [pud_alloc_one()][pud_alloc_one] which uses
[get_zeroed_page()][get_zeroed_page] to allocate the page from the physical
allocator.

A [memory barrier][memory-barrier] is established via [smp_wmb()][smp_wmb] to
ensure that the potentially newly allocated page is in its correct state prior
to being made visible by other CPUs after being placed into the page tables.

After this, the `mm->page_table_lock` spinlock is taken, and in the critical
section the PGD entry is checked to see whether it has been filled behind our
back - if so the PUD page is freed via [pud_free()][pud_free].

Otherwise, the PGD entry is populated via [pgd_populate()][pgd_populate] which
subsequently calls [set_pgd()][set_pgd] on the [__pgd()][__pgd]-created PGD
entry value obtaining the physical address of the new PUD page via
[__pa()][__pa], and assigning the bitfield of page flags defined by
[_PAGE_TABLE][_PAGE_TABLE]. [native_set_pgd()][native_set_pgd] finally assigns
this value in the PGD entry.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] within which the page table mappings
  live.

* `pgd` - The PGD entry which is to point at the newly allocated PUD page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PUD page for.

#### Returns

A pointer to the PUD entry in the possibly newly allocated PUD page the
specified PGD entry points at. This entry may be empty.

---

### pud_free()

`void pud_free(struct mm_struct *mm, pud_t *pud)`

[pud_free()][pud_free] frees the specified PUD page `pud` via
[free_page()][free_page], after first checking that `pud` is page-aligned.

#### Arguments

* `mm` (ignored) - The [struct mm_struct][mm_struct] the PUD page belongs to.

* `pud` - The PUD page we want to free.

#### Returns

N/A

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

* `pgd` - PGD entry whose pointed at PUD page is desired.

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

* `pud` - PUD entry whose pointed at PMD page is desired.

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

* `pmd` - PMD entry whose pointed at PTE page is desired.

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

* `pgd` - PGD entry whose pointed at PUD page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PGD entry's
pointed at PUD page.

---

### pud_page()

`struct page *pud_page(pud_t pud)`

[pud_page()][pud_page] returns the virtual address of the [struct page][page] of
the PMD pointed to by the specified PUD entry. This is a bit confusing, as
`pud_page()` returns the [struct page][page] for a PMD, not a PUD.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pud` - PUD entry whose pointed at PMD page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PUD entry's
pointed at PMD page.

---

### pmd_page()

`struct page *pmd_page(pmd_t pmd)`

[pmd_page()][pmd_page] returns the virtual address of the [struct page][page] of
the PTE pointed to by the specified PMD entry. This is a bit confusing, as
`pmd_page()` returns the [struct page][page] for a PTE, not a PMD.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pmd` - PMD entry whose pointed at PTE page's [struct page][page] is desired.

#### Returns

The virtual address of the [struct page][page] describing the PMD entry's
pointed at PTE page.

---

### pte_page()

`struct page *pte_page(pte_t pte)`

[pte_page()][pte_page] returns the virtual address of the [struct page][page] of
the physical page pointed to by the specified PTE entry. This is a bit
confusing, as `pte_page()` returns the [struct page][page] for the ultimate
physical page, not a PTE.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pte` - PTE entry whose pointed at physical page's [struct page][page] is
  desired.

#### Returns

The virtual address of the [struct page][page] describing the PTE entry's
pointed at physical page.

---

### pgd_none()

`int pgd_none(pgd_t pgd)`

[pgd_none()][pgd_none] determines whether the specified PGD entry is empty.

#### Arguments

* `pgd` - PGD entry which we want to determine is empty or not.

#### Returns

1 if the PGD entry is empty, 0 if not.

---

### pud_none()

`int pud_none(pud_t pud)`

[pud_none()][pud_none] determines whether the specified PUD entry is empty.

#### Arguments

* `pud` - PUD entry which we want to determine is empty or not.

#### Returns

1 if the PUD entry is empty, 0 if not.

---

### pmd_none()

`int pmd_none(pmd_t pmd)`

[pmd_none()][pmd_none] determines whether the specified PMD entry is empty.

#### Arguments

* `pmd` - PMD entry which we want to determine is empty or not.

#### Returns

1 if the PMD entry is empty, 0 if not.

---

### pte_none()

`int pte_none(pte_t pte)`

[pte_none()][pte_none] determines whether the specified PTE entry is empty.

#### Arguments

* `pte` - PTE entry which we want to determine is empty or not.

#### Returns

1 if the PTE entry is empty, 0 if not.

---

### pgd_bad()

`int pgd_bad(pgd_t pgd)`

[pgd_bad()][pgd_bad] determines whether the specified PGD entry itself is not in
a state where it, or descendent tables, can be safely modified.

__NOTE:__ It seems to me that it also determines whether a page table entry is
in a consistent state _at all_, since page tables that are empty/swapped
out/read-only make no sense for the purposes of page table traversal (discussed
below), but based on the definition from
[Understanding the Linux Virtual Memory Manager][amazon-gorman] and what I've
found online so far, 'modifiability' seems to be the canonical explanation so
far, so I'm being conservative and sticking with that. On other architectures it
seems the definition is more nebulous, I briefly
[looked into][arm64-stackoverflow] recently where the definition was - does this
entry actually refer to a lower level table.

In x86-64 the test consists of checking that the `_PAGE_PRESENT`, `_PAGE_RW`,
`_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set.

It's important to keep in mind that the PGD entry, if non-empty, contains the
physical address/swap metadata for the page that contains a corresponding PUD.

If the page isn't present (e.g., but not only, swapped out) then modifying the
entry itself will overwrite metadata, adjusting flags on the assumption the
entry is a physical address may result in invalid state, and trying to traverse
the entry in order to modify a lower level entry will not work since the entry
is not a physical address.

If the page isn't set dirty/accessed this would imply that the PUD page contains
no entries, which would make no sense - a PGD entry pointing to an empty PUD
indicates inconsistent state.

Finally, if the page is read-only then clearly its descendant entries cannot be
modified and presumably the read-only provision indicates that the entry's flags
should also not be adjusted (other than code that is knowingly and intentionally
removing the read-only status, which will not be performing a `pgd_bad()`
check.)

#### Arguments

* `pgd` - PGD entry we either (potentially) intend to modify or whose
  descendants we (potentially) intend to modify.

#### Returns

Truthy (non-zero) if the PGD entry or its descendants are unsafe to modify.

---

### pud_bad()

`int pud_bad(pud_t pud)`

[pud_bad()][pud_bad] determines whether the specified PUD entry itself is not in
a state where it, or descendent tables, can be safely modified.

__NOTE:__ It seems to me that it also determines whether a page table entry is
in a consistent state _at all_, since page tables that are marked empty/swapped
out/read-only make no sense for the purposes of page table traversal (discussed
below), but based on the definition from
[Understanding the Linux Virtual Memory Manager][amazon-gorman] and what I've
found online so far, 'modifiability' seems to be the canonical explanation so
far, so I'm being conservative and sticking with that. On other architectures it
seems the definition is more nebulous, I briefly
[looked into][arm64-stackoverflow] recently where the definition was - does this
entry actually refer to a lower level table.

In x86-64 the test consists of checking that the `_PAGE_PRESENT`, `_PAGE_RW`,
`_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set.

It's important to keep in mind that the PUD entry, if non-empty, contains the
physical address/swap metadata for the page that contains a corresponding PMD.

If the page isn't present (e.g., but not only, swapped out) then modifying the
entry itself will overwrite metadata, adjusting flags on the assumption the
entry is a physical address may result in invalid state, and trying to traverse
the entry in order to modify a lower level entry will not work since the entry
is not a physical address.

If the page isn't set dirty/accessed this would imply that the PMD page contains
no entries, which would make no sense - a PUD entry pointing to an empty PMD
indicates inconsistent state.

Finally, if the page is read-only then clearly its descendant entries cannot be
modified and presumably the read-only provision indicates that the entry's flags
should also not be adjusted (other than code that is knowingly and intentionally
removing the read-only status, which will not be performing a `pud_bad()`
check.)

#### Arguments

* `pud` - PUD entry we either (potentially) intend to modify or whose
  descendants we (potentially) intend to modify.

#### Returns

Truthy (non-zero) if the PUD entry or its descendants are unsafe to modify.

---

### pmd_bad()

`int pmd_bad(pmd_t pmd)`

[pmd_bad()][pmd_bad] determines whether the specified PMD entry itself is not in
a state where it, or descendent tables, can be safely modified.

__NOTE:__ It seems to me that it also determines whether a page table entry is
in a consistent state _at all_, since page tables that are marked empty/swapped
out/read-only make no sense for the purposes of page table traversal (discussed
below), but based on the definition from
[Understanding the Linux Virtual Memory Manager][amazon-gorman] and what I've
found online so far, 'modifiability' seems to be the canonical explanation so
far, so I'm being conservative and sticking with that. On other architectures it
seems the definition is more nebulous, I briefly
[looked into][arm64-stackoverflow] recently where the definition was - does this
entry actually refer to a lower level table.

In x86-64 the test consists of checking that the `_PAGE_PRESENT`, `_PAGE_RW`,
`_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set.

It's important to keep in mind that the PMD entry, if non-empty, contains the
physical address/swap metadata for the page that contains a corresponding PTE.

If the page isn't present (e.g., but not only, swapped out) then modifying the
entry itself will overwrite metadata, adjusting flags on the assumption the
entry is a physical address may result in invalid state, and trying to traverse
the entry in order to modify a lower level entry will not work since the entry
is not a physical address.

If the page isn't set dirty/accessed this would imply that the PTE page contains
no entries, which would make no sense - a PMD entry pointing to an empty PTE
indicates inconsistent state.

Finally, if the page is read-only then clearly its descendant entries cannot be
modified and presumably the read-only provision indicates that the entry's flags
should also not be adjusted (other than code that is knowingly and intentionally
removing the read-only status, which will not be performing a `pmd_bad()`
check.)

#### Arguments

* `pmd` - PMD entry we either (potentially) intend to modify or whose
  descendants we (potentially) intend to modify.

#### Returns

Truthy (non-zero) if the PMD entry or its descendants are unsafe to modify.

---

### pgd_flags()

`pgdval_t pgd_flags(pgd_t pgd)`

[pgd_flags()][pgd_flags] retrieves a bitfield containing the flags associated
with the specified PGD entry which, if the entry is non-empty, relate to the PUD
whose physical address (or swap metadata) is contained within the entry.

Typically this won't be accessed directly, rather a function wrapper for a
specific flag will be used.

#### Arguments

* `pgd` - PGD entry whose flags we desire.

#### Returns

A bitfield containing the PGD entry's flags.

---

### pud_flags()

`pudval_t pud_flags(pud_t pud)`

[pud_flags()][pud_flags] retrieves a bitfield containing the flags associated
with the specified PUD entry which, if the entry is non-empty, relate to the PMD
whose physical address (or swap metadata) is contained within the entry.

Typically this won't be accessed directly, rather a function wrapper for a
specific flag will be used.

#### Arguments

* `pud` - PUD entry whose flags we desire.

#### Returns

A bitfield containing the PUD entry's flags.

---

### pmd_flags()

`pmdval_t pmd_flags(pmd_t pmd)`

[pmd_flags()][pmd_flags] retrieves a bitfield containing the flags associated
with the specified PMD entry which, if the entry is non-empty, relate to the PTE
whose physical address (or swap metadata) is contained within the entry.

Typically this won't be accessed directly, rather a function wrapper for a
specific flag will be used.

#### Arguments

* `pmd` - PMD entry whose flags we desire.

#### Returns

A bitfield containing the PMD entry's flags.

---

### pte_flags()

`pteval_t pte_flags(pte_t pte)`

[pte_flags()][pte_flags] retrieves a bitfield containing the flags associated
with the specified PTE entry which, if the entry is non-empty, relate to the
physical page whose physical address (or swap metadata) is contained within the
entry.

Typically this won't be accessed directly, rather a function wrapper for a
specific flag will be used.

#### Arguments

* `pte` - PTE entry whose flags we desire.

#### Returns

A bitfield containing the PTE entry's flags.

---

### pud_pgprot

`pgprot_t pud_pgprot(pud_t pud)`

[pud_pgprot()][pud_pgprot] retrieves the flags for the specified PUD entry via
[pud_flags][pud_flags] and converts them to a [pgprot_t][pgprot_t] via
[__pgprot()][__pgprot].

As with `pXX_t` the reason this has to exist is that `pgprot_t` is a `typedef
struct` used to enforce type safety - `typedef struct { pgprotval_t pgprot; }
pgprot_t;`.

#### Arguments

#### Returns

A [pgprot_t][pgprot_t] containing the specified PUD entry's flags.

---

### pmd_pgprot

`pgprot_t pmd_pgprot(pmd_t pmd)`

[pmd_pgprot()][pmd_pgprot] retrieves the flags for the specified PMD entry via
[pmd_flags][pmd_flags] and converts them to a [pgprot_t][pgprot_t] via
[__pgprot()][__pgprot].

As with `pXX_t` the reason this has to exist is that `pgprot_t` is a `typedef
struct` used to enforce type safety - `typedef struct { pgprotval_t pgprot; }
pgprot_t;`.

#### Arguments

#### Returns

A [pgprot_t][pgprot_t] containing the specified PMD entry's flags.

---

### pte_pgprot

`pgprot_t pte_pgprot(pte_t pte)`

[pte_pgprot()][pte_pgprot] retrieves the flags for the specified PTE entry via
[pte_flags][pte_flags] and converts them to a [pgprot_t][pgprot_t] via
[__pgprot()][__pgprot].

As with `pXX_t` the reason this has to exist is that `pgprot_t` is a `typedef
struct` used to enforce type safety - `typedef struct { pgprotval_t pgprot; }
pgprot_t;`.

#### Arguments

#### Returns

A [pgprot_t][pgprot_t] containing the specified PTE entry's flags.

---

### __pgprot()

`pgprot_t __pgprot(pgprotval_t val)`

[__pgprot()][__pgprot] wraps the specified [pgprotval_t][pgprotval_t] (`unsigned
long` in x86-64) into a [pgprot_t][pgprot_t] structure.

As with `pXX_t` the reason this has to exist is that `pgprot_t` is a `typedef
struct` used to enforce type safety - `typedef struct { pgprotval_t pgprot; }
pgprot_t;`.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `val` - The [pgprotval_t][pgprotval_t] value which we want to translate into a
  [pgprot_t][pgprot_t].

#### Returns

A [pgprot_t][pgprot_t] containing the specified [pgprotval_t][pgprotval_t]
value.

---

### massage_pgprot()

`pgprotval_t massage_pgprot(pgprot_t pgprot)`

[massage_pgprot()][massage_pgprot] determines the value of the specified
[pgprot_t][pgprot_t] argument via [pgprot_val()][pgprot_val] and checks whether
the page is present (i.e. has the `_PAGE_PRESENT` flag set) - if so, it returns
the [pgprotval_t][pgprotval_t] of the flags masked against all possible flag
values.

If the page is not present, then no masking takes place. As the code comment
says, those flags can be used for other purposes when the page is not present so
it is not appropriate to modify them in that case.

#### Arguments

* `pgprot` - The [pgprot_t][pgprot_t] value which needs to be masked against
  possible flag values.

#### Returns

A [pgprotval_t][pgprotval_t] containing the specified flag bitfield masked
against all valid flags.

---

### canon_pgprot()

`pgprot_t canon_pgprot(pgprot_t pgprot)`

[canon_pgprot()][canon_pgprot] does the same task as
[massage_pgprot()][massage_pgprot], however it uses [__pgprot()][__pgprot] to
wrap the [pgprotval_t][pgprotval_t] returned by `massage_pgprot()` into a new
[pgprot_t][pgprot_t].

See the `massage_pgprot()` definition above for more details on what it
achieves.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pgprot` - The [pgprot_t][pgprot_t] value which needs to be masked against
  possible flag values.

#### Returns

A [pgprot_t][pgprot_t] containing the specified flag bitfield masked against all
valid flags.

---

### pmd_set_flags()

`pmd_t pmd_set_flags(pmd_t pmd, pmdval_t set)`

[pmd_set_flags()][pmd_set_flags] returns a new [pmd_t][pmd_t] which is identical
to the specified PMD entry except for the specified `set` field which is
'appended' (i.e. bitwise-or'd) to the input PMD entry's flag bitfield.

No checks are performed to ensure `set` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pmd` - The PMD entry which we want to copy/update.

* `set` - The flag bitfield we want to append to the newly created PMD entry.

#### Returns

A copy of the input PMD entry with `set` appended to its flag bitfield.

---

### pte_set_flags()

`pte_t pte_set_flags(pte_t pte, pteval_t set)`

[pte_set_flags()][pte_set_flags] returns a new [pte_t][pte_t] which is identical
to the specified PTE entry except for the specified `set` field which is
'appended' (i.e. bitwise-or'd) to the input PTE entry's flag bitfield.

No checks are performed to ensure `set` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pte` - The PTE entry which we want to copy/update.

* `set` - The flag bitfield we want to append to the newly created PTE entry.

#### Returns

A copy of the input PTE entry with `set` appended to its flag bitfield.

---

### pmd_clear_flags()

`pmd_t pmd_clear_flags(pmd_t pmd, pmdval_t clear)`

[pmd_clear_flags()][pmd_clear_flags] returns a new [pmd_t][pmd_t] which is
identical to the specified PMD entry except for the specified `clear` field
whose bitfield is cleared in the returned PMD entry's flag bitfield.

No checks are performed to ensure `clear` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pmd` - The PMD entry which we want to copy/update.

* `clear` - The flag bitfield we want to clear from the newly created PMD entry.

#### Returns

A copy of the input PMD entry with `clear` flags cleared in its flag bitfield.

---

### pte_clear_flags()

`pte_t pte_clear_flags(pte_t pte, pteval_t clear)`

[pte_clear_flags()][pte_clear_flags] returns a new [pte_t][pte_t] which is
identical to the specified PTE entry except for the specified `clear` field
whose bitfield is cleared in the returned PTE entry's flag bitfield.

No checks are performed to ensure `clear` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pte` - The PTE entry which we want to copy/update.

* `clear` - The flag bitfield we want to clear from the newly created PTE entry.

#### Returns

A copy of the input PTE entry with `clear` flags cleared in its flag bitfield.

---

### pgd_present()

`int pgd_present(pgd_t pgd)`

[pgd_present()][pgd_present] determines whether the PUD page that the specified
PGD entry points at is 'present' - i.e. whether it is currently resident in
memory and not swapped out or otherwise unavailable.

The function uses [pgd_flags()][pgd_flags] to retrieve the PGD entry's flags
then tests whether `_PAGE_PRESENT` is set.

#### Arguments

* `pgd` - PGD entry pointing at the PUD page which we want to determine is
  present or not.

#### Returns

Truthy (non-zero) if the PUD page is present, 0 if not.

---

### pud_present()

`int pud_present(pud_t pud)`

[pud_present()][pud_present] determines whether the PMD page that the specified
PUD entry points at is 'present' - i.e. whether it is currently resident in
memory and not swapped out or otherwise unavailable.

The function uses [pud_flags()][pud_flags] to retrieve the PUD entry's flags
then tests whether `_PAGE_PRESENT` is set.

#### Arguments

* `pud` - PUD entry pointing at the PMD page which we want to determine is
  present or not.

#### Returns

Truthy (non-zero) if the PMD page is present, 0 if not.

---

### pmd_present()

`int pmd_present(pmd_t pmd)`

[pmd_present()][pmd_present] determines whether the PTE page that the specified
PMD entry points at is 'present' - i.e. whether it is currently resident in
memory and not swapped out or otherwise unavailable.

The function uses [pmd_flags()][pmd_flags] to retrieve the PMD entry's flags
then tests whether `_PAGE_PRESENT`, `_PAGE_PROTNONE` or `_PAGE_PSE` are set.

Looking at each flag:

* `_PAGE_PRESENT` indicates whether the pointed at PTE page is actually resident
  or not.

* `_PAGE_PROTNONE` indicates that the PTE page is resident, but not
  accessible. This is used by NUMA balancing (irrelevant for our assumed
  architecture, UMA x86-64) or the [mprotect][mprotect] `PROT_NONE` flag having
  been set by the user.

* `_PAGE_PSE` indicates that huge pages are in use, and needs to be checked here
  because (according to the comment for the [pmd_present()][pmd_present]
  function) the [split_huge_page()][split_huge_page] function will temporarily
  clear the present bit, so we need a means of determining whether the page is
  still actually present when this happens.

The `_PAGE_PROTNONE` and `_PAGE_PSE` flags seem only to be meaningful at the PMD
level when the PTE is a huge page (i.e. 2MiB in x86-64), which seems only to be
the case when either [transparent huge pages][transhuge] or the
[device mapper][device-mapper] are in use.

#### Arguments

* `pmd` - PMD entry pointing at the PTE page which we want to determine is
  present or not.

#### Returns

Truthy (non-zero) if the PTE page is present, 0 if not.

---

### pte_present()

`int pte_present(pte_t pte)`

[pte_present()][pte_present] determines whether the physical page that the
specified PTE entry points at is 'present' - i.e. whether it is currently
resident in memory and not swapped out or otherwise unavailable.

The function uses [pte_flags()][pte_flags] to retrieve PTE flags then tests
whether `_PAGE_PRESENT` or `_PAGE_PROTNONE` are set.

Looking at each flag:

* `_PAGE_PRESENT` is the flag that indicates whether the physical page is
  actually resident or not.

* `_PAGE_PROTNONE` indicates that the page is resident, but not accessible. This
  is used by NUMA balancing (irrelevant for our assumed architecture, UMA
  x86-64) or the [mprotect][mprotect] `PROT_NONE` flag having been set by the
  user.

#### Arguments

* `pte` - PTE entry pointing at the physical page which we want to determine is
  present or not.

#### Returns

Truthy (non-zero) if the physical page is present, 0 if not.

---

### pmd_young()

`int pmd_young(pmd_t pmd)`

[pmd_young()][pmd_young] determines whether the specified PMD entry has the
`_PAGE_ACCESSED` flag set, i.e. whether the PTE page it refers to has been
accessed since the flag was last cleared.

#### Arguments

* `pmd` - The PMD entry whose accessed flag state we want to determine.

#### Returns

Truthy (non-zero) if the PMD entry is marked accessed.

---

### pte_young()

`int pte_young(pte_t pte)`

[pte_young()][pte_young] determines whether the specified PTE entry has the
`_PAGE_ACCESSED` flag set, i.e. whether the physical page it refers to has been
accessed since the flag was last cleared.

#### Arguments

* `pte` - The PTE entry whose accessed flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked accessed.

---

### pmd_dirty()

`int pmd_dirty(pmd_t pmd)`

[pmd_dirty()][pmd_dirty] determines whether the specified PMD entry has the
`_PAGE_DIRTY` flag set, i.e. whether the PTE page it refers to has been modified
since the flag was last cleared.

This function can only be meaningfully used if `pmd_present()` returns true.

#### Arguments

* `pmd` - The PMD entry whose dirty flag state we want to determine.

#### Returns

Truthy (non-zero) if the PMD entry is marked dirty.

---

### pte_dirty()

`int pte_dirty(pte_t pte)`

[pte_dirty()][pte_dirty] determines whether the specified PTE entry has the
`_PAGE_DIRTY` flag set, i.e. whether the physical page it refers to has been
modified since the flag was last cleared.

This function can only be meaningfully used if `pte_present()` returns true.

#### Arguments

* `pte` - The PTE entry whose dirty flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked dirty.

---

### pmd_write()

`int pmd_write(pmd_t pmd)`

[pmd_write()][pmd_write] determines whether the specified PMD entry has the
`_PAGE_RW` flag set, i.e. whether the PTE page it refers to is _not_ read-only.

#### Arguments

* `pmd` - The PMD entry whose read/write flag state we want to determine.

#### Returns

Truthy (non-zero) if the PMD entry is marked read/write.

---

### pte_write()

`int pte_write(pte_t pte)`

[pte_write()][pte_write] determines whether the specified PTE entry has the
`_PAGE_RW` flag set, i.e. whether the physical page it refers to is _not_
read-only.

#### Arguments

* `pte` - The PTE entry whose read/write flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked read/write.

---

### pte_exec()

`int pte_exec(pte_t pte)`

[pte_exec()][pte_exec] determines whether the specified PTE entry has the
`_PAGE_NX` flag set, i.e. whether the physical page it refers to contains bytes
that can be executed.

This uses the [NX bit][nx-bit] feature of x86-64 which determines whether bytes
in a specific page are permitted to be executed or not.

#### Arguments

* `pte` - The PTE entry whose executable flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked executable.

---

### pte_global()

`int pte_global(pte_t pte)`

[pte_global()][pte_global] determines whether the specified PTE entry has the
`_PAGE_GLOBAL` flag set, i.e. whether the [TLB][tlb] cache entry mapping the
PTE's corresponding virtual address to the physical page it points at ought to
_not_ to be cleared when the TLB is flushed, either manually or by a context
switch.

This flag is useful for pages which are shared across all processes and are
regularly accessed.

Once set, TLB flushes caused by a task switch (i.e. assigning a new PGD to the
`cr3` register) or a manual TLB flush other than
[flush_tlb_all()][flush_tlb_all] will not invalidate the global entry.

If you want to finally invalidate such an entry, a call to
[__flush_tlb_global()][__flush_tlb_global] and subsequently
[invpcid_flush_all()][invpcid_flush_all] is required (this will most likely be
via [flush_tlb_all()][flush_tlb_all].)

#### Arguments

* `pte` - The PTE entry whose global flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked global.

---

### pte_special()

`int pte_special(pte_t pte)`

[pte_special()][pte_special] determines whether the specified PTE entry has the
`_PAGE_SPECIAL` flag set, i.e. whether the 'user-defined' special flag is set.

I put 'user-defined' in quotes to avoid confusion between user and kernel - here
user is in the sense of the software rather than the CPU. The
`_PAGE_BIT_SPECIAL` constant (`_PAGE_SPECIAL = 1UL << _PAGE_BIT_SPECIAL`) is an
alias for `_PAGE_BIT_SOFTW1` - bit 9 (base-0), first of the 3 bits between 9 and
11 which the CPU simply ignores.

#### Arguments

* `pte` - The PTE entry whose special flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked special.

---

### pmd_soft_dirty()

`int pmd_soft_dirty(pmd_t pmd)`

[pmd_soft_dirty()][pmd_soft_dirty] determines whether the specified PMD entry
has the `_PAGE_SOFT_DIRTY` flag set, i.e. whether the 'user-defined' soft-dirty
flag is set.

I put 'user-defined' in quotes to avoid confusion between user and kernel - here
user is in the sense of the software rather than the CPU. The
`_PAGE_BIT_SOFT_DIRTY` constant (`_PAGE_SOFT_DIRTY = 1UL <<
_PAGE_BIT_SOFT_DIRTY`) is an alias for `_PAGE_BIT_SOFTW3` - bit 11 (base-0),
third of the 3 bits between 9 and 11 which the CPU simply ignores.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

Since the flags field describes the underlying PTE page, this function
determines whether the _PTE page_ has been modified since last use, despite the
'PMD' in its name.

#### Arguments

* `pmd` - The PMD entry whose soft-dirty flag state we want to determine.

#### Returns

Truthy (non-zero) if the PMD entry is marked soft-dirty.

---

### pte_soft_dirty()

`int pte_soft_dirty(pte_t pte)`

[pte_soft_dirty()][pte_soft_dirty] determines whether the specified PTE entry
has the `_PAGE_SOFT_DIRTY` flag set, i.e. whether the 'user-defined' soft-dirty
flag is set.

I put 'user-defined' in quotes to avoid confusion between user and kernel - here
user is in the sense of the software rather than the CPU. The
`_PAGE_BIT_SOFT_DIRTY` constant (`_PAGE_SOFT_DIRTY = 1UL <<
_PAGE_BIT_SOFT_DIRTY`) is an alias for `_PAGE_BIT_SOFTW3` - bit 11 (base-0),
third of the 3 bits between 9 and 11 which the CPU simply ignores.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

Since the flags field describes the underlying physical page page, this function
determines whether the referred to _physical page_ has been modified since last
use, despite the 'PTE' in its name.

#### Arguments

* `pte` - The PTE entry whose soft-dirty flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked soft-dirty.

---

### pmd_devmap()

`int pmd_devmap(pmd_t pmd)`

[pmd_devmap()][pmd_devmap] determines whether the specified PMD entry has the
`_PAGE_DEVMAP` flag set, i.e. whether the 'user-defined' devmap flag is set.

I put 'user-defined' in quotes to avoid confusion between user and kernel - here
user is in the sense of the software rather than the CPU. The `_PAGE_BIT_DEVMAP`
constant (`_PAGE_DEVMAP = 1UL << _PAGE_BIT_DEVMAP`) is an alias for
`_PAGE_BIT_SOFTW4` - bit 58 (base-0), which the CPU simply ignores.

It seems like this flag forms part of the [device mapper][device-mapper]
functionality but I haven't yet looked into this deeply.

#### Arguments

* `pmd` - The PMD entry whose devmap flag state we want to determine.

#### Returns

Truthy (non-zero) if the PMD entry is marked as a devmap entry.

---

### pte_devmap()

`int pte_devmap(pte_t pte)`

[pte_devmap()][pte_devmap] determines whether the specified PTE entry has the
`_PAGE_DEVMAP` flag set, i.e. whether the 'user-defined' devmap flag is set.

I put 'user-defined' in quotes to avoid confusion between user and kernel - here
user is in the sense of the software rather than the CPU. The `_PAGE_BIT_DEVMAP`
constant (`_PAGE_DEVMAP = 1UL << _PAGE_BIT_DEVMAP`) is an alias for
`_PAGE_BIT_SOFTW4` - bit 58 (base-0), which the CPU simply ignores.

It seems like this flag forms part of the [device mapper][device-mapper]
functionality but I haven't yet looked into this deeply.

#### Arguments

* `pte` - The PTE entry whose devmap flag state we want to determine.

#### Returns

Truthy (non-zero) if the PTE entry is marked as a devmap entry.

---

### pud_huge()

`int pud_huge(pud_t pud)`

[pud_huge()][pud_huge] determines whether the specified PUD entry is marked huge
in the context of [hugetlb][hugetlb], i.e. whether the PMD page it refers to is
huge under the `hugetlb` scheme.

This simply checks the `_PAGE_PSE` (i.e. huge page) flag.

#### Arguments

* `pud` - The PUD entry whose huge flag state we want to determine.

#### Returns

Truthy (non-zero) if the PUD entry is marked huge.

---

### pmd_huge()

`int pmd_huge(pmd_t pmd)`

[pmd_huge()][pmd_huge] determines whether the specified PMD entry is marked huge
in the context of [hugetlb][hugetlb], i.e. whether the PTE page it refers to is
huge under the `hugetlb` scheme.

This function determines whether the specified PMD entry is non-empty and either
not marked present or marked present and has the `_PAGE_PSE` (i.e. huge page)
flag set.

#### Arguments

* `pmd` - The PMD entry we want to determine is marked huge under the `hugetlb`
  scheme.

#### Returns

Truthy (non-zero) if the PMD entry is marked huge under the `hugetlb` scheme.

---

### pte_huge()

`int pte_huge(pte_t pte)`

[pte_huge()][pte_huge] determines whether the specified PTE entry is marked
huge, i.e. whether the physical page it refers to is huge.

This simply checks the `_PAGE_PSE` (i.e. huge page) flag.

#### Arguments

* `pte` - The PTE entry we want to determine is marked huge or not.

#### Returns

Truthy (non-zero) if the PTE entry is marked huge.

---

### pmd_trans_huge()

`int pmd_trans_huge(pmd_t pmd)`

[pmd_trans_huge()][pmd_trans_huge] determines whether the specified PMD entry is
marked huge in the context of [transparent huge pages][transhuge].

This checks that the `_PAGE_PSE` flag is set and the `_PAGE_DEVMAP` flag is
_not_ set.

#### Arguments

* `pmd` - The PMD entry we want to determine is marked huge or not under the
  Transparent Huge Pages scheme.

#### Returns

Truthy (non-zero) if the PMD entry is marked huge under the Transparent Huge
Pages scheme.

---

### pgd_large()

`int pgd_large(pgd_t pgd)`

[pgd_large()][pgd_large] determines whether the specified PGD entry is marked
huge, indicating the PUD page it points at is huge.

This function is defined as returning 0 on x86-64 - PUD pages are never huge.

#### Arguments

* `pgd` - The PGD entry we want to determine is marked huge or not.

#### Returns

Truthy (non-zero) if the PGD entry is marked huge.

---

### pud_large()

`int pud_large(pud_t pud)`

[pud_large()][pud_large] determines whether the specified PUD entry is marked
huge, indicating the PMD page it points at is huge.

The function tests the `_PAGE_PSE` and `_PAGE_PRESENT` flags to determine
whether the page is marked huge and not swapped out/otherwise unavailable,
respectively.

#### Arguments

* `pud` - The PUD entry we want to determine is marked huge or not.

#### Returns

Truthy (non-zero) if the PUD entry is marked huge.

---

### pmd_large()

`int pmd_large(pmd_t pmd)`

[pmd_large()][pmd_large] determines whether the specified PMD entry is marked
huge, indicating the PTE page it points at is huge.

The function simply tests the `_PAGE_PSE` flag to determine whether the page is
marked huge. It differs from [pud_large()][pud_large] in that it doesn't also
check for the present flag, an odd inconsistency.

#### Arguments

* `pmd` - The PMD entry we want to determine is marked huge or not.

#### Returns

Truthy (non-zero) if the PMD entry is marked huge.

---

### pmd_mkyoung()

`pmd_t pmd_mkyoung(pmd_t pmd)`

[pmd_mkyoung()][pmd_mkyoung] returns a new [pmd_t][pmd_t] which is identical to
the specified PMD entry except that the `_PAGE_ACCESSED` flag is set, indicating
that the underlying PTE page has been accessed.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the accessed flag set.

#### Returns

A copy of the input PMD entry with the accessed flag set.

---

### pmd_mkold()

`pmd_t pmd_mkold(pmd_t pmd)`

[pmd_mkold()][pmd_mkold] returns a new [pmd_t][pmd_t] which is identical to the
specified PMD entry except that the `_PAGE_ACCESSED` flag is cleared, indicating
that the underlying PTE page has not been accessed yet.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the accessed flag cleared.

#### Returns

A copy of the input PMD entry with the accessed flag cleared.

---

### pte_mkyoung()

`pte_t pte_mkyoung(pte_t pte)`

[pte_mkyoung()][pte_mkyoung] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_ACCESSED` flag is set, indicating
that the underlying physical page has been accessed.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the accessed flag set.

#### Returns

A copy of the input PTE entry with the accessed flag set.

---

### pte_mkold()

`pte_t pte_mkold(pte_t pte)`

[pte_mkold()][pte_mkold] returns a new [pte_t][pte_t] which is identical to the
specified PTE entry except that the `_PAGE_ACCESSED` flag is cleared, indicating
that the underlying physical page has not been accessed yet.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the accessed flag cleared.

#### Returns

A copy of the input PTE entry with the accessed flag cleared.

---

### pmd_mkdirty()

`pmd_t pmd_mkdirty(pmd_t pmd)`

[pmd_mkdirty()][pmd_mkdirty] returns a new [pmd_t][pmd_t] which is identical to
the specified PMD entry except that the `_PAGE_DIRTY` and `_PAGE_SOFT_DIRTY`
flags are set, indicating that the underlying PTE page has been modified.

The `_PAGE_SOFT_DIRTY` flag being set synchronises the [soft-dirty][soft-dirty]
state with the kernel page ageing logic.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the dirty flags set.

#### Returns

A copy of the input PMD entry with the dirty flags set.

---

### pmd_mkclean()

`pmd_t pmd_mkclean(pmd_t pmd)`

[pmd_mkclean()][pmd_mkclean] returns a new [pmd_t][pmd_t] which is identical to
the specified PMD entry except that the `_PAGE_DIRTY` flag is cleared,
indicating that the underlying PTE page has not been modified yet.

Curiously, the `_PAGE_SOFT_DIRTY` flag is not cleared (it's curious because this
flag _is_ set in [pmd_mkdirty()][pmd_mkdirty].)

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the dirty flag cleared.

#### Returns

A copy of the input PMD entry with the dirty flag cleared.

---

### pte_mkdirty()

`pte_t pte_mkdirty(pte_t pte)`

[pte_mkdirty()][pte_mkdirty] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_DIRTY` and `_PAGE_SOFT_DIRTY`
flags are set, indicating that the underlying physical page has been modified.

The `_PAGE_SOFT_DIRTY` flag being set synchronises the [soft-dirty][soft-dirty]
state with the kernel page ageing logic.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the dirty flags set.

#### Returns

A copy of the input PTE entry with the dirty flags set.

---

### pte_mkclean()

`pte_t pte_mkclean(pte_t pte)`

[pte_mkclean()][pte_mkclean] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_DIRTY` flag is cleared,
indicating that the underlying physical page has not been dirty yet.

Curiously, the `_PAGE_SOFT_DIRTY` flag is not cleared (it's curious because this
flag _is_ set in [pte_mkdirty()][pte_mkdirty].)

#### Arguments

* `pte` - The PTE entry which we want a copy of with the dirty flag cleared.

#### Returns

A copy of the input PTE entry with the dirty flag cleared.

---

### pmd_mkwrite()

`pmd_t pmd_mkwrite(pmd_t pmd)`

[pmd_mkwrite()][pmd_mkwrite] returns a new [pmd_t][pmd_t] which is identical to
the specified PMD entry except that the `_PAGE_RW` flag is set, indicating that
the underlying PTE page can be read from and written to.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the R/W flag set.

#### Returns

A copy of the input PMD entry with the R/W flag set.

---

### pmd_wrprotect()

`pmd_t pmd_wrprotect(pmd_t pmd)`

[pmd_wrprotect()][pmd_wrprotect] returns a new [pmd_t][pmd_t] which is identical
to the specified PMD entry except that the `_PAGE_RW` flag is cleared,
indicating that the underlying PTE page is read-only.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the R/W flag cleared.

#### Returns

A copy of the input PMD entry with the R/W flag cleared.

---

### pte_mkwrite()

`pte_t pte_mkwrite(pte_t pte)`

[pte_mkwrite()][pte_mkwrite] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_RW` flag is set, indicating that
the underlying physical page can be read from and written to.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the R/W flag set.

#### Returns

A copy of the input PTE entry with the R/W flag set.

---

### pte_wrprotect()

`pte_t pte_wrprotect(pte_t pte)`

[pte_wrprotect()][pte_wrprotect] returns a new [pte_t][pte_t] which is identical to the
specified PTE entry except that the `_PAGE_RW` flag is cleared, indicating that
the underlying physical page is read-only.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the R/W flag cleared.

#### Returns

A copy of the input PTE entry with the R/W flag cleared.

---

### pmd_mksoft_dirty()

`pmd_t pmd_mksoft_dirty(pmd_t pmd)`

[pmd_mksoft_dirty()][pmd_mksoft_dirty] returns a new [pmd_t][pmd_t] which is
identical to the specified PMD entry except that the `_PAGE_SOFT_DIRTY` flag is
set, indicating that the underlying PTE page has been modified since the
soft-dirty flag was last set.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the soft-dirty flag set.

#### Returns

A copy of the input PMD entry with the soft-dirty flag set.

---

### pmd_clear_soft_dirty()

`pmd_t pmd_clear_soft_dirty(pmd_t pmd)`

[pmd_clear_soft_dirty()][pmd_clear_soft_dirty] returns a new [pmd_t][pmd_t]
which is identical to the specified PMD entry except that the `_PAGE_SOFT_DIRTY`
flag is cleared, indicating that the underlying PTE page has not been modified
since the soft-dirty flag was last set.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the soft-dirty flag
  cleared.

#### Returns

A copy of the input PMD entry with the soft-dirty flag cleared.

---

### pte_mksoft_dirty()

`pte_t pte_mksoft_dirty(pte_t pte)`

[pte_mksoft_dirty()][pte_mksoft_dirty] returns a new [pte_t][pte_t] which is
identical to the specified PTE entry except that the `_PAGE_SOFT_DIRTY` flag is
set, indicating that the underlying physical page has been modified since the
soft-dirty flag was last set.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the soft-dirty flag set.

#### Returns

A copy of the input PTE entry with the soft-dirty flag set.

---

### pte_clear_soft_dirty()

`pte_t pte_clear_soft_dirty(pte_t pte)`

[pte_clear_soft_dirty()][pte_clear_soft_dirty] returns a new [pte_t][pte_t]
which is identical to the specified PTE entry except that the `_PAGE_SOFT_DIRTY`
flag is cleared, indicating that the underlying physical page has not been
modified since the soft-dirty flag was last set.

The [soft-dirty][soft-dirty] mechanism is a means by which userland processes
can determine which pages have been modified since these bits were last cleared
for a specified task via the `/proc/<pid>/clear_refs` and `/proc/<pid>/pagemap`
files.

It differs from the actual dirty flag as that is used by the kernel for its own
modified state tracking, rendering it unsuitable for use by userland.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the soft-dirty flag cleared.

#### Returns

A copy of the input PTE entry with the soft-dirty flag cleared.

---

### pmd_mkdevmap()

`pmd_t pmd_mkdevmap(pmd_t pmd)`

[pmd_mkdevmap()][pmd_mkdevmap] returns a new [pmd_t][pmd_t] which is identical
to the specified PMD entry except that the `_PAGE_DEVMAP` flag is set,
indicating that the underlying PTE page is used by the
[device mapping][device-mapper] functionality.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the dev-map flag set.

#### Returns

A copy of the input PMD entry with the dev-map flag set.

---

### pte_mkdevmap()

`pte_t pte_mkdevmap(pte_t pte)`

[pte_mkdevmap()][pte_mkdevmap] returns a new [pte_t][pte_t] which is identical
to the specified PTE entry except that the `_PAGE_DEVMAP` flag is set,
indicating that the underlying physical page is used by the
[device mapping][device-mapper] functionality.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the dev-map flag set.

#### Returns

A copy of the input PTE entry with the dev-map flag set.

---

### pmd_mknotpresent()

`pmd_t pmd_mknotpresent(pmd_t pmd)`

[pmd_mknotpresent()][pmd_mknotpresent] returns a new [pmd_t][pmd_t] which is
identical to the specified PMD entry except that the `_PAGE_PRESENT` and
`_PAGE_PROTNONE` flags are cleared, indicating that the underlying PTE page is
not resident, not even as a non-readable page.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the present flags cleared.

#### Returns

A copy of the input PMD entry with the present flags cleared.

---

### pte_mkglobal()

`pte_t pte_mkglobal(pte_t pte)`

[pte_mkglobal()][pte_mkglobal] returns a new [pte_t][pte_t] which is identical
to the specified PTE entry except that the `_PAGE_GLOBAL` flag is set,
indicating that the [TLB][tlb] cache entry mapping the PTE's corresponding
virtual address to the physical page it points at _won't_ be cleared when the
TLB is flushed, either manually or by a context switch.

This flag is useful for pages which are shared across all processes and are
regularly accessed.

Once set, TLB flushes caused by a task switch (i.e. assigning a new PGD to the
`cr3` register) or a manual TLB flush other than
[flush_tlb_all()][flush_tlb_all] will not invalidate the global entry.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the global flag set.

#### Returns

A copy of the input PTE entry with the global flag set.

---

### pte_clrglobal()

`pte_t pte_clrglobal(pte_t pte)`

[pte_clrglobal()][pte_clrglobal] returns a new [pte_t][pte_t] which is identical
to the specified PTE entry except that the `_PAGE_GLOBAL` flag is cleared,
indicating that the [TLB][tlb] cache entry mapping the PTE's corresponding
virtual address to the physical page it points at ought to be cleared when the
TLB is flushed, either manually or by a context switch (when the flag is set,
this does _not_ happen.)

#### Arguments

* `pte` - The PTE entry which we want a copy of with the global flag cleared.

#### Returns

A copy of the input PTE entry with the global flag cleared.

---

### pte_mkexec()

`pte_t pte_mkexec(pte_t pte)`

[pte_mkexec()][pte_mkexec] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_NX` flag is cleared, indicating
that the underlying physical page contains executable code.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the no-execute flag cleared.

#### Returns

A copy of the input PTE entry with the no-execute flag cleared.

---

### pte_mkspecial()

`pte_t pte_mkspecial(pte_t pte)`

[pte_mkspecial()][pte_mkspecial] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_SPECIAL` flag is set, indicating
that the underlying physical page is special.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the special flag set.

#### Returns

A copy of the input PTE entry with the special flag set.

---

### pmd_mkhuge()

`pmd_t pmd_mkhuge(pmd_t pmd)`

[pmd_mkhuge()][pmd_mkhuge] returns a new [pmd_t][pmd_t] which is identical to
the specified PMD entry except that the `_PAGE_PSE` flag is set, indicating that
the underlying PTE page is huge.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the huge flag set.

#### Returns

A copy of the input PMD entry with the huge flag set.

---

### pte_mkhuge()

`pte_t pte_mkhuge(pte_t pte)`

[pte_mkhuge()][pte_mkhuge] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_PSE` flag is set, indicating that
the underlying physical page is huge (i.e. 2MiB in x86-64.)

#### Arguments

* `pte` - The PTE entry which we want a copy of with the huge flag set.

#### Returns

A copy of the input PTE entry with the huge flag set.

---

### pte_clrhuge()

`pte_t pte_clrhuge(pte_t pte)`

[pte_clrhuge()][pte_clrhuge] returns a new [pte_t][pte_t] which is identical to the
specified PTE entry except that the `_PAGE_PSE` flag is cleared, indicating
that the underlying physical page is not huge.

#### Arguments

* `pte` - The PTE entry which we want a copy of with the huge flag cleared.

#### Returns

A copy of the input PTE entry with the huge flag cleared.

---

[CONFIG_HIGHPTE]:http://cateee.net/lkddb/web-lkddb/HIGHPTE.html
[PGALLOC_GFP]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L9
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[_PAGE_TABLE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L118
[__flush_tlb_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L63
[__get_free_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L500
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__pgd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L407
[__pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L74
[__pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L363
[__pmd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L500
[__pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L83
[__pte/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L377
[__pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L87
[__pud/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L540
[__pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L78
[__pud_alloc]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3544
[_pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L343
[_pgd_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L348
[amazon-gorman]:http://www.amazon.co.uk/Understanding-Virtual-Memory-Manager-Perens/dp/0131453483
[arm64-stackoverflow]:http://stackoverflow.com/a/37433195/6380063
[bitmask]:https://en.wikipedia.org/wiki/Mask_(computing)
[canon_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L431
[clone_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L879
[device-mapper]:https://en.wikipedia.org/wiki/Device_mapper
[flush_tlb_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L280
[free_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L520
[get_zeroed_page]:https://github.com/torvalds/linux/blob/v4.6/mm/page_alloc.c#L3421
[high-memory]:https://en.wikipedia.org/wiki/High_memory
[hugetlb]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/hugetlbpage.txt
[invpcid_flush_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L48
[massage_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L372
[memory-barrier]:https://en.wikipedia.org/wiki/Memory_barrier
[mm_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mprotect]:http://man7.org/linux/man-pages/man2/mprotect.2.html
[native_make_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L254
[native_make_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L293
[native_make_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L347
[native_make_pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L272
[native_pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L259
[native_pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L298
[native_pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L352
[native_pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L277
[native_set_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L109
[nx-bit]:https://en.wikipedia.org/wiki/NX_bit
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[paravirtualisation]:https://en.wikipedia.org/wiki/Paravirtualization
[pfn_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L388
[pfn_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L382
[pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L354
[pgd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L689
[pgd_ctor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L116
[pgd_dtor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L136
[pgd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L264
[pgd_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L394
[pgd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L708
[pgd_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L130
[pgd_list_add]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L87
[pgd_list_del]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L94
[pgd_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L694
[pgd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L714
[pgd_offset_k]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L719
[pgd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L676
[pgd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L667
[pgd_populate]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L120
[pgd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L662
[pgd_set_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L105
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pgd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L421
[pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L73
[pgdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pgprot_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L250
[pgprot_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L362
[pgprotval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L16
[pmd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L605
[pmd_clear_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L283
[pmd_clear_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L361
[pmd_devmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L190
[pmd_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L121
[pmd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L342
[pmd_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L337
[pmd_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L62
[pmd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L575
[pmd_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L173
[pmd_mkclean]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L295
[pmd_mkdevmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L310
[pmd_mkdirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L305
[pmd_mkhuge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L315
[pmd_mknotpresent]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L330
[pmd_mkold]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L290
[pmd_mksoft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L351
[pmd_mkwrite]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L325
[pmd_mkyoung]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L320
[pmd_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L550
[pmd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L639
[pmd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L566
[pmd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L557
[pmd_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L161
[pmd_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L329
[pmd_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L428
[pmd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L521
[pmd_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L276
[pmd_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L341
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmd_trans_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L179
[pmd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L514
[pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L82
[pmd_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L850
[pmd_wrprotect]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L300
[pmd_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L126
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pte_clear_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L204
[pte_clear_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L356
[pte_clrglobal]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L261
[pte_clrhuge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L251
[pte_devmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L497
[pte_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L97
[pte_exec]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L146
[pte_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L357
[pte_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L141
[pte_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L136
[pte_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L595
[pte_lockptr]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1619
[pte_mkclean]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L211
[pte_mkdevmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L271
[pte_mkdirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L231
[pte_mkexec]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L226
[pte_mkglobal]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L256
[pte_mkhuge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L246
[pte_mkold]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L216
[pte_mksoft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L346
[pte_mkspecial]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L266
[pte_mkwrite]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L241
[pte_mkyoung]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L236
[pte_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L480
[pte_offset_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L600
[pte_offset_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L140
[pte_offset_map_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1680
[pte_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L171
[pte_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L156
[pte_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L427
[pte_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L491
[pte_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L197
[pte_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L336
[pte_special]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L151
[pte_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pte_unmap_unlock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1689
[pte_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L393
[pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L86
[pte_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L131
[pte_wrprotect]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L221
[pte_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L116
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[pud_alloc]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1576
[pud_alloc_one]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L126
[pud_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L650
[pud_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L324
[pud_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L319
[pud_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L131
[pud_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L68
[pud_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L679
[pud_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L644
[pud_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L616
[pud_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L684
[pud_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L635
[pud_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L626
[pud_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L166
[pud_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L311
[pud_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L429
[pud_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L621
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pud_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L554
[pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L77
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[set_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L56
[smp_wmb]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/barrier.h#L84
[soft-dirty]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/soft-dirty.txt
[split_huge_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/huge_mm.h#L92
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
