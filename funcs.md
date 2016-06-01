# VM Functions

__Certainty Level:__ Moderate, am trying to check functions as best I can, but
might be making mistakes.

__Architecture:__ These notes assume an x86-64 UMA system unless otherwise
specified. [Linux 4.6][linux-4.6] is always targeted.

## Contents

### Address Translation

* [phys_to_virt()](#phys_to_virt) - Translates a physical address to a virtual
  one.
* [virt_to_phys()](#virt_to_phys) - Translates a virtual address to a physical
  one.
* [virt_to_page()](#virt_to_page) - Retrieves the [struct page][page] that
  describes the page containing the specified virtual address.
* [page_to_pfn()](#page_to_pfn) - Converts a [struct page][page] to its
  corresponding Page Frame Number (PFN.)
* [page_to_phys()](#page_to_phys) - Retrieve the physical address of the page
  described by the specified [struct page][page].
* [pfn_to_page()](#pfn_to_page) - Converts a Page Frame Number (PFN) to its
  corresponding [struct page][page].
* [__va()](#__va) - Translates a physical address to a virtual one.
* [__pa()](#__pa) - Translates a virtual address to a physical one.

### Page Tables

#### Retrieving Page Table Entries

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

#### Retrieving Page Table Entry Values

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

#### Masking Page Table Entry Values

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

#### Creating Page Table Entries

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

#### Page Table Entry State

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

#### Retrieving Flag Bitfields

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

#### Creating Flag Bitfields

* [__pgprot()](#__pgprot) - Converts the specified [pgprotval_t][pgprotval_t]
  into a [pgprot_t][pgprot_t].
* [massage_pgprot()](#massage_pgprot) - Masks the specified [pgprot_t][pgprot_t]
  fields against all possible flags, returns [pgprotval_t][pgprotval_t].
* [canon_pgprot()](#canon_pgprot) -  Masks the specified [pgprot_t][pgprot_t]
  fields against all possible flags, returns [pgprot_t][pgprot_t].

#### Setting/Clearing Flag Bitfields

* [pmd_set_flags()](#pmd_set_flags) - Returns a new copy of a PMD with the
  specified bitfield appended to its flag bitfield.
* [pte_set_flags()](#pte_set_flags) - Returns a new copy of a PTE with the
  specified bitfield appended to its flag bitfield.
* [pmd_clear_flags()](#pmd_clear_flags) - Returns a new copy of a PMD with the
  specified bitfield cleared from its flag bitfield.
* [pte_clear_flags()](#pte_clear_flags) - Returns a new copy of a PTE with the
  specified bitfield cleared from its flag bitfield.

#### Retrieving Individual Flags

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

##### Huge Pages

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

#### Utility Functions

* [virt_addr_valid()](#virt_addr_valid) - Determines if a specified virtual
  address is a valid _kernel_ virtual address.
* [pfn_valid()](#pfn_valid) - Determines if a specified Page Frame Number (PFN)
  represents a valid physical address.
* [pte_same()](#pte_same) - Determines whether the two specified PTE entries
  refer to the same physical page and flags.

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

[__va()][__va] does the heavy lifting for `phys_to_virt()`, converting a
physical memory address to a kernel-valid virtual one.

This function simply adds the kernel memory offset `PAGE_OFFSET`,
`0xffff880000000000` for x86-64. Looking at the [x86-64 memory map][x86-64-mm],
you can see this simply provides a virtual address that is part of the kernel's
_complete_ direct mapping of all physical memory between `0xffff880000000000`
and `0xffffc7ffffffffff`.

The function performs no checks and assumes the supplied physical memory address
is valid and in the 64TiB of allowed memory in current x86-64 linux kernel
implementations.

__NOTE:__ Macro, inferring function signature.

---

### __pa()

`phys_addr_t __pa(volatile void *address)`

[__pa()][__pa] does the heavy lifting for `virt_to_phys()`, converting a
_kernel_ virtual memory address to a physical one. The function is a wrapper
around [__phys_addr()][__phys_addr].

The function isn't quite as simple as [__va()][__va] as it has to determine
whether the supplied virtual address is part of the kernel's direct mapping of
_all_ of the physical memory between `0xffff880000000000` and
`0xffffc7ffffffffff`, or whether it's part of the kernel's 'text' from
`__START_KERNEL_map` on (`0xffffffff80000000` for [x86-64][x86-64-mm]), and
offsets the supplied virtual address accordingly.

In the case of the address originating from the kernel's 'text', the physical
offset of the kernel, `phys_base` is taken into account, which allows for the
kernel to be loaded in a different physical location e.g. when
[kdump][kdump]ing.

__NOTE:__ Macro, inferring function signature.

---

### virt_to_page()

`struct page *virt_to_page(unsigned long kaddr)`

[virt_to_page()][virt_to_page] determines the physical address of the specified
kernel virtual address, then the Page Frame Number (PFN) of the physical page
that contains it, and finally passes this to [pfn_to_page()][pfn_to_page] to
retrieve the [struct page][page] which describes the physical page.

__Important:__ As per the code comment above the function, the returned pointer
is valid if and only if [virt_addr_valid(kaddr)][virt_addr_valid] returns
`true`.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `kaddr` - The virtual _kernel_ address whose [struct page][page] we desire.

#### Returns

The [struct page][page] describing the physical page the specified virtual
address resides in.

---

### page_to_pfn()

`unsigned long page_to_pfn(struct page *page)`

[page_to_pfn()][page_to_pfn] returns the Page Frame Number (PFN) that is
associated with the specified [struct page][page].

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

How the PFN is determined varies depending on the memory model, in x86-64 UMA
this is [__page_to_pfn()][__page_to_pfn] under `CONFIG_SPARSEMEM_VMEMMAP` - the
memory map is virtually contiguous at `vmemmap`, (`0xffffea0000000000`, see
[x86-64 memory map][x86-64-mm].)

This makes the implementation of the function straightforward - simply subtract
`vmemmap` from the page pointer (being careful with typing to have pointer
arithmetic take into account `sizeof(struct page)` for you.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `page` - The [struct page][page] whose corresponding Page Frame Number (PFN)
  is desired.

#### Returns

The PFN of the specified [struct page][page].

---

### page_to_phys()

`dma_addr_t page_to_phys(struct page *page)`

[page_to_phys()][page_to_phys] returns a physical address for the page described
by the specified [struct page][page].

Oddly it seems to return a [dma_addr_t][dma_addr_t], possibly due to use for
device I/O (it is declared in `arch/x86/include/asm/io.h`), however in x86-64 it
makes no difference as `dma_addr_t` and [phys_addr_t][phys_addr_t] are the
same - unsigned long (64-bit.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `page` - The [struct page][page] whose physical start address we desire.

#### Returns

The physical address of the start of the page which the specified
[struct page][page] describes.

---

### pfn_to_page()

`struct page *pfn_to_page(unsigned long pfn)`

[pfn_to_page()][pfn_to_page] returns the [struct page][page] that is associated
with the specified Page Frame Number (PFN.)

The PFN of a physical address is simply the (masked) address's value shifted
right by the number of bits of the page size, so in a standard x86-64
configuration, 12 bits (equivalent to the default 4KiB page size), and `pfn =
masked_phys_addr >> 12`.

How the [struct page][page] is located varies depending on the memory model, in
x86-64 UMA this is [__pfn_to_page()][__pfn_to_page] under
`CONFIG_SPARSEMEM_VMEMMAP` - the memory map is virtually contiguous at
`vmemmap`, (`0xffffea0000000000`, see [x86-64 memory map][x86-64-mm].)

This makes the implementation of the function straightforward - simply offset
the PFN by `vmemmap` (being careful with typing to have pointer arithmetic take
into account `sizeof(struct page)` for you.)

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pfn` - The Page Frame Number (PFN) whose corresponding [struct page][page] is
  desired.

#### Returns

The [struct page][page] that describes the physical page with specified PFN.

## Page Tables

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
to the specified PMD except for the specified `set` field which is 'appended'
(i.e. bitwise-or'd) to the input PMD's flag bitfield.

No checks are performed to ensure `set` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pmd` - The PMD which we want to copy/update.

* `set` - The flag bitfield we want to append to the newly created PMD.

#### Returns

A copy of the input PMD with `set` appended to its flag bitfield.

---

### pte_set_flags()

`pte_t pte_set_flags(pte_t pte, pteval_t set)`

[pte_set_flags()][pte_set_flags] returns a new [pte_t][pte_t] which is identical
to the specified PTE except for the specified `set` field which is 'appended'
(i.e. bitwise-or'd) to the input PTE's flag bitfield.

No checks are performed to ensure `set` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pte` - The PTE which we want to copy/update.

* `set` - The flag bitfield we want to append to the newly created PTE.

#### Returns

A copy of the input PMD with `set` appended to its flag bitfield.

---

### pmd_clear_flags()

`pmd_t pmd_clear_flags(pmd_t pmd, pmdval_t clear)`

[pmd_clear_flags()][pmd_clear_flags] returns a new [pmd_t][pmd_t] which is
identical to the specified PMD except for the specified `clear` field whose
bitfield is cleared in the returned PMD's flag bitfield.

No checks are performed to ensure `clear` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pmd` - The PMD which we want to copy/update.

* `clear` - The flag bitfield we want to clear from the newly created PMD.

#### Returns

A copy of the input PMD with `clear` flags cleared in its flag bitfield.

---

### pte_clear_flags()

`pte_t pte_clear_flags(pte_t pte, pteval_t clear)`

[pte_clear_flags()][pte_clear_flags] returns a new [pte_t][pte_t] which is
identical to the specified PTE except for the specified `clear` field whose
bitfield is cleared in the returned PTE's flag bitfield.

No checks are performed to ensure `clear` is valid and limited only to available
flag bits, so incorrect use of this function could result in an invalid return
value.

#### Arguments

* `pte` - The PTE which we want to copy/update.

* `clear` - The flag bitfield we want to clear from the newly created PTE.

#### Returns

A copy of the input PTE with `clear` flags cleared in its flag bitfield.

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
be cleared when the TLB is flushed, either manually or by a context switch.

This flag is useful for pages which are shared across all processes and are
regularly accessed.

Once set, TLB flushes caused by a task switch (i.e. assigning a new PGD to the
`cr3` register) or a call to [__flush_tlb()][__flush_tlb] (which simply reads
then writes back `cr3`) will not invalidate the global entry.

If you want to finally invalidate such an entry, a call to
[__flush_tlb_global()][__flush_tlb_global] and subsequently
[invpcid_flush_all()][invpcid_flush_all] is required.

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

### virt_addr_valid()

`bool virt_addr_valid(unsigned long kaddr)`

[virt_addr_valid()][virt_addr_valid] determines if the specified virtual address
`kaddr` is actually a valid, non-[vmalloc][vmalloc]'d kernel address.

The function is a wrapper for [__virt_addr_valid()][__virt_addr_valid], which,
once its checked the virtual address is in a valid range, checks it has a valid
corresponding physical PFN via [pfn_valid()][pfn_valid].

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `kaddr` - Virtual address which we want to determine is a valid non-vmalloc'd
  kernel address or not.

#### Returns

`true` if the address is valid, `false` if not.

---

### pfn_valid()

`int pfn_valid(unsigned long pfn)`

[pfn_valid()][pfn_valid] determine whether the specified Page Frame Number (PFN)
is valid, i.e. in x86-64 whether it refers to a valid 46-bit address, and
whether there is actually physical memory mapped to that physical location.

__NOTE:__ Macro, inferring function signature.

#### Arguments

* `pfn` - PFN whose validity we wish to determine.

#### Returns

Truthy (non-zero) if the PFN is valid, 0 if not.

---

### pte_same()

`int pte_same(pte_t a, pte_t b)`

[pte_same()][pte_same] determines whether the two specified PTE entries refer to
the same physical page AND share the same flags.

On x86-64 it's as simple as `a.pte == b.pte`.

#### Arguments

* `a` - The first PTE entry whose physical page address and flags we want to
  compare.

* `b` - The second PTE entry whose physical page address and flags we want to
  compare.

#### Returns

1 if the PTE entries' physical address and flags are the same, 0 if not.

---

[linux-4.6]:https://github.com/torvalds/linux/tree/v4.6/

[pgdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L15
[pudval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L14
[pmdval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L13
[pteval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L12
[pgprotval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L16
[pgprot_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L250
[bitmask]:https://en.wikipedia.org/wiki/Mask_(computing)
[page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L44
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[soft-dirty]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/soft-dirty.txt
[hugetlb]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/hugetlbpage.txt
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt

[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
[__phys_addr]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page_64.h#L26
[kdump]:https://github.com/torvalds/linux/blob/v4.6/Documentation/kdump/kdump.txt
[virt_to_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L63
[page_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L144
[dma_addr_t]:https://github.com/torvalds/linux/blob/v4.6/include/linux/types.h#L152
[phys_addr_t]:https://github.com/torvalds/linux/blob/v4.6/include/linux/types.h#L162

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
[pgprot_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L362
[pud_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L166
[pmd_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L161
[pte_pfn]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L156
[pud_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L311
[pmd_pfn_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L329
[pud_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L319
[pmd_flags_mask]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L337
[native_make_pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L254
[native_make_pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L272
[native_make_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L293
[native_make_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L347
[__pgd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L74
[__pgd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L407
[__pud]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L78
[__pud/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L540
[__pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L83
[__pmd/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L500
[__pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L87
[__pte/para]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/paravirt.h#L377
[pfn_pmd]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L388
[massage_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L372
[pfn_pte]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L382
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
[pgd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L264
[pud_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L324
[pmd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L342
[pte_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L357
[pud_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L429
[pmd_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L428
[pte_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L427
[__pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L363
[canon_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L431
[pmd_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L276
[pmd_clear_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L283
[pte_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L197
[pte_clear_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L204
[pgd_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L694
[pud_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L616
[pmd_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L550
[pte_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L480
[pgd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L662
[pud_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L621
[pmd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L521
[mprotect]:http://man7.org/linux/man-pages/man2/mprotect.2.html
[split_huge_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/huge_mm.h#L92
[device-mapper]:https://en.wikipedia.org/wiki/Device_mapper
[pte_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L491
[pgd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L689
[amazon-gorman]:http://www.amazon.co.uk/Understanding-Virtual-Memory-Manager-Perens/dp/0131453483
[arm64-stackoverflow]:http://stackoverflow.com/a/37433195/6380063
[pud_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L650
[pmd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L605
[pmd_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L126
[pte_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L116
[pmd_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L121
[pte_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L97
[pud_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L68
[pmd_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L62
[pte_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L136
[pmd_trans_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L179
[pgd_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64.h#L130
[pud_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L644
[pmd_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L173
[pmd_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L850
[pte_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L131
[pte_exec]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L146
[nx-bit]:https://en.wikipedia.org/wiki/NX_bit
[pte_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L141
[__flush_tlb]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L62
[__flush_tlb_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L63
[invpcid_flush_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L48
[pte_special]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L151
[pmd_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L341
[pte_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L336
[pmd_devmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L190
[pte_devmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L497
[pfn_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L81
[__pfn_to_page]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L53
[page_to_pfn]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L80
[__page_to_pfn]:https://github.com/torvalds/linux/blob/v4.6/include/asm-generic/memory_model.h#L54
[virt_addr_valid]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L66
[vmalloc]:http://www.makelinux.net/books/lkd2/ch11lev1sec5
[__virt_addr_valid]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/physaddr.c#L86
[pfn_valid]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1140
[pte_same]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L486
