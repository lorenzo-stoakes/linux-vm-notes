# Page Table Functions

## Contents

### Flag Functions

See [Page Table Flag Functions][page-table-flag-funcs], which lists
functions relating to page table entry flags.

These have been moved for readability.

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
  specified address and returns a pointer to its PUD entry.
* [pud_free()](#pud_free) - Frees the specified PUD page.
* [pmd_alloc()](#pmd_alloc) - If necessary, allocates a new PMD page for the
  specified address and returns a pointer to its PMD entry.
* [pmd_free()](#pmd_free) - Frees the specified PMD page.
* [pte_alloc()](#pte_alloc) - If necessary, allocates a new PTE page.
* [pte_alloc_map()](#pte_alloc_map) - If necessary, allocates a new PTE page for
  the specified address and returns its PTE entry pointer.
* [pte_alloc_map_lock()](#pte_alloc_map_lock) - [pte_alloc_map()][pte_alloc_map]
  and acquires a lock on the PTE.
* [pte_alloc_kernel()](#pte_alloc_kernel) - Allocates a PTE for a kernel
  mapping.
* [pte_free()](#pte_free) - Frees the specified PTE page.
* [pte_free_kernel()](#pte_free_kernel) - Frees the specified kernel-mapped PTE
  page.

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

---

## Function Descriptions

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

```c
pte_t *pte_offset_map_lock(struct mm_struct *mm, pmd_t *pmd, unsigned long address,
                           spinlock_t **ptlp)
```

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

Note that the below _only_ applies if the mapping does not already exist and
allocation is required.

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

Note that [split page table locks][split-page-table-lock] do not provide
fine-grained locks for page tables above PMDs, and since we're locking at the
PGD level here we use the [struct mm_struct][mm_struct]'s lock.

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

### pmd_alloc()

`pmd_t *pmd_alloc(struct mm_struct *mm, pud_t *pud, unsigned long address)`

[pmd_alloc()][pmd_alloc] first checks to see whether the specified PUD entry
`pud` is empty, if so it allocates a new PMD page and modifies the PUD entry to
point at that page, otherwise it leaves the existing mapping in place. It then
returns the PMD entry indexed by the virtual `address` within that page.

Note that the below _only_ applies if the mapping does not already exist and
allocation is required.

If allocation is required, this is performed via [__pmd_alloc()][__pmd_alloc]
which subsequently calls [pmd_alloc_one()][pmd_alloc_one] which uses
[alloc_pages()][alloc_pages] to allocate a physical [struct page][page] with the
`GFP_KERNEL`, `__GFP_REPEAT` and `__GFP_ZERO` flags set. Looking at the two
lower level flags:

* `__GFP_REPEAT` - Try hard to allocate the memory.
* `__GFP_ZERO` - Zero the page.

The returned PMD [struct page][page] is then passed to
[pgtable_pmd_page_ctor()][pgtable_pmd_page_ctor] which clears the
[transparent huge page][transhuge]-utilised field `pmd_huge_pte`, and
initialises the spinlock `page->ptl` via [ptlock_init()][ptlock_init] (this lock
is used by [split page table locks][split-page-table-lock].)

Finally the allocated page is translated to the returned virtual address via
[page_address()][page_address].

A [memory barrier][memory-barrier] is established via [smp_wmb()][smp_wmb] to
ensure that the potentially newly allocated page is in its correct state prior
to being made visible by other CPUs after being placed into the page tables.

After this, the `mm->page_table_lock` spinlock is taken, and in the critical
section the PUD entry is checked to see whether it has been filled behind our
back - if so the PMD page is freed via [pmd_free()][pmd_free].

Otherwise, the PUD entry is populated via [pud_populate()][pud_populate] which
subsequently calls [set_pud()][set_pud] on the [__pud()][__pud]-created PUD
entry value obtaining the physical address of the new PMD page via
[__pa()][__pa], and assigning the bitfield of page flags defined by
[_PAGE_TABLE][_PAGE_TABLE]. [native_set_pud()][native_set_pud] finally assigns
this value in the PUD entry.

Note that [split page table locks][split-page-table-lock] do not provide
fine-grained locks for page tables above PMDs, and since we're locking at the
PUD level here we use the [struct mm_struct][mm_struct]'s lock.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] within which the page table mappings
  live.

* `pud` - The PUD entry which is to point at the newly allocated PMD page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PMD page for.

#### Returns

A pointer to the PMD entry in the possibly newly allocated PMD page the
specified PUD entry points at. This entry may be empty.

---

### pmd_free()

`void pmd_free(struct mm_struct *mm, pmd_t *pmd)`

[pmd_free()][pmd_free] frees the specified PMD page `pmd`.

It first checks that `pmd` is page-aligned, then runs the destructor function
[pgtable_pmd_page_dtor()][pgtable_pmd_page_dtor] on the
[virt_to_page()][virt_to_page]-derived physical [struct page][page] associated
with the PMD page.

[pgtable_pmd_page_dtor()][pgtable_pmd_page_dtor] checks to ensure the
[transparent huge page][transhuge]-utilised `pmd_huge_pte` field is not set,
then frees the `page->ptl` spinlock allocated in [pmd_alloc()][pmd_alloc] via
[ptlock_free()][ptlock_free] (this lock is used by
[split page table locks][split-page-table-lock].)

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] the PMD page belongs to.

* `pmd` - The PMD page we want to free.

#### Returns

N/A

---

### pte_alloc()

`int pte_alloc(struct mm_struct *mm, pmd_t *pmd, unsigned long address)`

[pte_alloc()][pte_alloc] first checks to see whether the specified PMD entry
`pmd` is empty, if so it allocates a new PTE page and modifies the PMD entry to
point at that page, otherwise it leaves the existing mapping in place. It
returns 0 if no allocation was required or if the allocation succeeded, truthy
(non-zero) otherwise.

Note that the below _only_ applies if the mapping does not already exist and
allocation is required.

If allocation is required, this is performed via [__pte_alloc()][__pte_alloc]
which subsequently calls [pte_alloc_one()][pte_alloc_one] which uses
[alloc_pages()][alloc_pages] to allocate a physical [struct page][page] with the
`__userpte_alloc_gfp` bitfield which is equal to `PGALLOC_GFP |
PGALLOC_USER_GFP` which in x86-64 breaks down into `GFP_KERNEL` and:

* `__GFP_NOTRACK` - Avoid tracking with kmemcheck.
* `__GFP_REPEAT` - Try hard to allocate the memory.
* `__GFP_ZERO` - Zero the page.

The returned PTE [struct page][page] is then passed to
[pgtable_page_ctor()][pgtable_page_ctor] which initialises the spinlock
`page->ptl` via [ptlock_init()][ptlock_init] and invokes
[inc_zone_page_state()][inc_zone_page_state] to increment the `NR_PAGETABLE`
statistic for the current NUMA zone.

Interestingly, unlike [pmd_alloc_one()][pmd_alloc_one],
[pte_alloc_one()][pte_alloc_one] returns a [pgtable_t][pgtable_t] (typedef'd to
[struct page *][page].)

A [memory barrier][memory-barrier] is established via [smp_wmb()][smp_wmb] to
ensure that the potentially newly allocated page is in its correct state prior
to being made visible by other CPUs after being placed into the page tables.

After this, [pmd_lock()][pmd_lock] is invoked to acquire and return a PMD-level
spinlock. This functionality is made possible by kernel
[split page table locks][split-page-table-lock], which provide more
finely-grained locking than simply acquiring
[struct mm_struct][mm_struct]`->page_table_lock`:

[pmd_lock()][pmd_lock] invokes [pmd_lockptr()][pmd_lockptr] on the supplied
[struct mm_struct][mm_struct] and PMD entry [pmd_t *][pmd_t] to retrieve the
spinlock, before acquiring it and returning it. This returns the
[struct page][page]`->ptl` field (via [ptlock_ptr()][ptlock_ptr]) of the
[struct page][page] associated with the PMD page the specified `pmd` PMD entry
belongs to via [pmd_to_page()][pmd_to_page].

If the PMD entry has not been filled behind our back, we start the critical
section by incrementing [struct mm_struct][mm_struct]`->nr_ptes` fields.

We then populate the PMD entry via [pmd_populate()][pmd_populate] which
subsequently calls [set_pmd()][set_pmd] on the [__pmd()][__pmd]-created PMD
entry value obtaining the physical address of the new PTE page via
[__pa()][__pa], and assigning the bitfield of page flags defined by
[_PAGE_TABLE][_PAGE_TABLE]. [native_set_pmd()][native_set_pmd] finally assigns
this value in the PMD entry.

The spinlock is released and then, if the PMD entry had been filled behind our
back, we free the one we allocated via [pmd_free()][pmd_free].

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] within which the page table mappings
  live.

* `pmd` - The PMD entry which is to point at the newly allocated PTE page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PTE page for.

#### Returns

Truthy (non-zero) if the PTE allocation has failed, false (0) if no allocation
was required or the allocation succeeded.

---

### pte_alloc_map()

`pte_t *pte_alloc_map(struct mm_struct *mm, pmd_t *pmd, unsigned long address)`

[pte_alloc_map()][pte_alloc_map] invokes [pte_alloc()][pte_alloc] to allocate a
new PTE page if necessary, then uses [pte_offset_map()][pte_offset_map] to
return the PTE entry indexed by the specified virtual `address` in the possibly
new PTE page. See the descriptions of these functions for more details on their
operation.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] within which the page table mappings
  live.

* `pmd` - The PMD entry which is to point at the newly allocated PTE page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PTE page for.

#### Returns

A pointer to the PTE entry in the possibly newly allocated PTE page the
specified PMD entry points at. This entry may be empty.

---

### pte_alloc_map_lock()

```c
pte_t *pte_alloc_map_lock(struct mm_struct *mm, pmd_t *pmd, unsigned long address,
                          spinlock_t **ptlp)
```

[pte_alloc_map_lock()][pte_alloc_map_lock] performs the same task as
[pte_alloc_map()][pte_alloc_map], however it uses
[pte_offset_map_lock()][pte_offset_map_lock] to return the PTE entry rather than
[pte_offset_map()][pte_offset_map]. See the descriptions of these functions for
more details on their operation.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] within which the page table mappings
  live.

* `pmd` - The PMD entry which is to point at the newly allocated PTE page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PTE page for.

* `ptlp` - __OUT__ - A pointer to a `spinlock_t *` which will reference the
  spinlock acquired by the function.

#### Returns

A pointer to the PTE entry in the possibly newly allocated PTE page the
specified PMD entry points at. This entry may be empty.

---

### pte_alloc_kernel()

`pte_t *pte_alloc_kernel(pmd_t *pmd, unsigned long address)`

[pte_alloc_kernel()][pte_alloc_kernel] allocates a kernel PTE using the
[init_mm][init_mm] kernel [struct mm_struct][mm_struct] (and hence the
[swapper_pg_dir][swapper_pg_dir] PGD.)

The function operates similarly to [pte_alloc_map()][pte_alloc_map], only no
attempt at mapping PTEs is made (this makes no difference in x86-64 as there is
no high memory to worry about), amongst other differences. To avoid duplication,
let's look at these differences, you can obtain a description of the basic
functionality from the descriptions of [pte_alloc()][pte_alloc], etc. above.

The key differences are that the [init_mm][init_mm]`.page_table_lock` is used
rather than a finer-grained lock, and no statistics are updated in the
allocation.

The allocation, if required, is performed by
[pte_alloc_one_kernel()][pte_alloc_one_kernel] which simply invokes
[__get_free_page()][__get_free_page].

The PMD entry is populated via [pmd_populate_kernel()][pmd_populate_kernel]
which differs from [pmd_populate()][pmd_populate] in that it is parameterised on
a [pte_t][pte_t] rather than a [struct page][page].

If the PMD has been populated behind the functions back,
[pte_free_kernel()][pte_free_kernel] is used to free the allocated PTE page
rather than [pte_free()][pte_free]. This differs is that no statistics are
updated nor are finer-grained split lock attempted to be cleaned up.

Finally, the PTE entry indexed by `address` is returned via
[pte_offset_kernel()][pte_offset_kernel] which in x86-64 is equivalent to
[pte_offset_map()][pte_offset_map].

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pmd` - The PMD entry which is to point at the newly allocated PTE page (or
  whose mapping is to be used if already mapped.)

* `address` - The virtual address we are allocated the PTE page for.

#### Returns

A pointer to the PTE entry in the possibly newly allocated PTE page the
specified PMD entry points at. This entry may be empty.

---

### pte_free()

`void pte_free(struct mm_struct *mm, struct page *pte)`

[pte_free()][pte_free] frees the specified PTE [struct page][page] `pte`. This
function differs from other `pXX_free()` functions in that the physical
[struct page][page] describing the PTE is provided rather than a [pte_t][pte_t].

Firstly it invokes [pgtable_page_dtor()][pgtable_page_dtor] which calls
[pte_lock_deinit()][pte_lock_deinit] to set the [struct page][page]`->mapping`
field to NULL to avoid [free_pages_check()][free_pages_check] issue with
non-null mapping, and decrements the zone page statistics for `NR_PAGETABLE` via
[dec_zone_page_state()][dec_zone_page_state].

Finally the actual freeing of the PTE page is performed by
[__free_page()][__free_page].

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] the PTE page belongs to.

* `pte` - The physical [struct page][page] that describes the PTE page we want
  to free.

#### Returns

N/A

---

### pte_free_kernel()

`void pte_free_kernel(struct mm_struct *mm, pte_t *pte)`

[pte_free_kernel()][pte_free_kernel] frees the specified kernel mapping-only PTE
page `pte`, and performs no statistics update or split lock cleanup. It differs
from [pte_free()][pte_free] in that a [pte_t][pte_t] is input rather than a
[struct page][page], meaning it uses [free_page()][free_page] rather than
[__free_page()][__free_page].

#### Arguments

* `mm` - The [struct mm_struct][mm_struct] the PTE page belongs to. Can't see
  any reason why this will be anything other than `&`[init_mm][init_mm].

* `pte` - The PTE page we want to free.

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

[CONFIG_HIGHPTE]:http://cateee.net/lkddb/web-lkddb/HIGHPTE.html
[PGALLOC_GFP]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L9
[PTRS_PER_PGD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L28
[PTRS_PER_PMD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L41
[PTRS_PER_PTE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L46
[PTRS_PER_PUD]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L34
[_PAGE_TABLE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L118
[__free_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L519
[__get_free_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L500
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__pgd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L407
[__pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L74
[__pmd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L500
[__pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L83
[__pmd_alloc]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3567
[__pte/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L377
[__pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L87
[__pte_alloc]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L566
[__pud/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L540
[__pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L78
[__pud_alloc]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3544
[_pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L343
[_pgd_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L348
[alloc_pages]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L466
[bitmask]:https://en.wikipedia.org/wiki/Mask_(computing)
[clone_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L879
[dec_zone_page_state]:https://github.com/torvalds/linux/blob/v4.6/mm/vmstat.c#L377
[free_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/gfp.h#L520
[free_pages_check]:https://github.com/torvalds/linux/blob/v4.6/mm/page_alloc.c#L787
[get_zeroed_page]:https://github.com/torvalds/linux/blob/v4.6/mm/page_alloc.c#L3421
[high-memory]:https://en.wikipedia.org/wiki/High_memory
[inc_zone_page_state]:https://github.com/torvalds/linux/blob/v4.6/mm/vmstat.c#L371
[init_mm]:https://github.com/torvalds/linux/blob/v4.6/mm/init-mm.c#L16
[massage_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L372
[memory-barrier]:https://en.wikipedia.org/wiki/Memory_barrier
[mm_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[native_make_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L254
[native_make_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L293
[native_make_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L347
[native_make_pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L272
[native_pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L259
[native_pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L298
[native_pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L352
[native_pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L277
[native_set_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L109
[native_set_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L63
[native_set_pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L99
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[page_address]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L986
[paravirtualisation]:https://en.wikipedia.org/wiki/Paravirtualization
[pfn_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L388
[pfn_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L382
[pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L354
[pgd_ctor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L116
[pgd_dtor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L136
[pgd_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L394
[pgd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L708
[pgd_list_add]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L87
[pgd_list_del]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L94
[pgd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L714
[pgd_offset_k]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L719
[pgd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L676
[pgd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L667
[pgd_populate]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L120
[pgd_set_mm]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L105
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pgd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L421
[pgd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L73
[pgdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pgprot_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L250
[pgprot_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L362
[pgprotval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L16
[pgtable_page_ctor]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1666
[pgtable_page_dtor]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1674
[pgtable_pmd_page_ctor]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1721
[pgtable_pmd_page_dtor]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1729
[pgtable_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L417
[pmd_alloc]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1582
[pmd_alloc_one]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L81
[pmd_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L337
[pmd_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L94
[pmd_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L575
[pmd_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1753
[pmd_lockptr]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1716
[pmd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L639
[pmd_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L566
[pmd_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L557
[pmd_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L161
[pmd_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L329
[pmd_populate]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L69
[pmd_populate_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L62
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmd_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1710
[pmd_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L514
[pmd_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L82
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pte_alloc]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1694
[pte_alloc_kernel]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1704
[pte_alloc_map]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1697
[pte_alloc_map_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1700
[pte_alloc_one]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L24
[pte_alloc_one_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L19
[pte_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L48
[pte_free_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L42
[pte_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L595
[pte_lock_deinit]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1641
[pte_lockptr]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1619
[pte_offset_kernel]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L600
[pte_offset_map]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L140
[pte_offset_map_lock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1680
[pte_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L171
[pte_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L156
[pte_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pte_unmap_unlock]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1689
[pte_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L393
[pte_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L86
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[ptlock_free]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L3974
[ptlock_init]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1624
[ptlock_ptr]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1613
[pud_alloc]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1576
[pud_alloc_one]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L126
[pud_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L319
[pud_free]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L131
[pud_index]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L679
[pud_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L684
[pud_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L635
[pud_page_vaddr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L626
[pud_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L166
[pud_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L311
[pud_populate]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgalloc.h#L112
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pud_val/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L554
[pud_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L77
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[set_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L56
[set_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L53
[set_pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L61
[smp_wmb]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/barrier.h#L84
[split-page-table-lock]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/split_page_table_lock
[swapper_pg_dir]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L25
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
[virt_to_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L63

[page-table-flag-funcs]:./page-table-flag-funcs.md
