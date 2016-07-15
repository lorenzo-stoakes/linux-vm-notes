# Page Table Flag Functions

## Contents

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
* [pmd_huge()](#pmd_huge) - Determines if the specified PMD entry points at a
  huge physical page in the context of [hugetlb][hugetlb].
* [pte_huge()](#pte_huge) - Determines if the physical page pointed at by the
  specified PTE entry is huge (without context.)
* [pmd_trans_huge()](#pmd_trans_huge) - Determines if the specified PMD entry
  points at a huge physical page in the context of
  [transparent huge pages][transhuge].
* [pud_large()](#pud_large) - Determines if the specified PUD entry points at a
  gigantic physical page.
* [pmd_large()](#pmd_large) - Determines if the specified PMD entry points at a
  huge physical page.

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

## Function Descriptions

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
a state where it, or descendent tables, can be safely modified or referenced.

In x86-64 the test consists of masking out `_PAGE_USER` (ignored so the test can
be applied to both kernel and userland mappings), then checking that the
`_PAGE_PRESENT`, `_PAGE_RW`, `_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set,
and __only__ those flags are set. If this is not the case, then the entry is
bad.

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
a state where it, or descendent tables, can be safely modified or referenced.

In x86-64 the test consists of masking out `_PAGE_USER` (ignored so the test can
be applied to both kernel and userland mappings), then checking that the
`_PAGE_PRESENT`, `_PAGE_RW`, `_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set,
and __only__ those flags are set. If this is not the case, then the entry is
bad.

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
a state where it, or descendent tables, can be safely modified or referenced.

In x86-64 the test consists of masking out `_PAGE_USER` (ignored so the test can
be applied to both kernel and userland mappings), then checking that the
`_PAGE_PRESENT`, `_PAGE_RW`, `_PAGE_ACCESSED` and `_PAGE_DIRTY` flags are set,
and __only__ those flags are set. If this is not the case, then the entry is
bad.

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
level when it refers to a huge page (i.e. 2MiB in x86-64), which seems only to
be the case when either [transparent huge pages][transhuge] or the
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
(2MiB on x86-64) in the context of [hugetlb][hugetlb], i.e. whether the PMD page
it refers to is huge under the `hugetlb` scheme.

This simply checks the `_PAGE_PSE` (i.e. huge page) flag.

#### Arguments

* `pud` - The PUD entry whose huge flag state we want to determine.

#### Returns

Truthy (non-zero) if the PUD entry is marked huge.

---

### pmd_huge()

`int pmd_huge(pmd_t pmd)`

[pmd_huge()][pmd_huge] determines whether the specified PMD entry is marked huge
(2MiB on x86-64) in the context of [hugetlb][hugetlb], i.e. whether it points at
a huge physical page rather than an ordinary PTE page, under the `hugetlb`
scheme.

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

[pte_huge()][pte_huge] determines whether the specified PTE entry is marked huge
(2MiB on x86-64), i.e. whether the physical page it refers to is huge.

Typically this will be a PMD that is cast to a PTE.

This simply checks the `_PAGE_PSE` (i.e. huge page) flag.

#### Arguments

* `pte` - The PTE entry we want to determine is marked huge or not.

#### Returns

Truthy (non-zero) if the PTE entry is marked huge.

---

### pmd_trans_huge()

`int pmd_trans_huge(pmd_t pmd)`

[pmd_trans_huge()][pmd_trans_huge] determines whether the specified PMD entry is
marked huge (2MiB on x86-64) in the context of
[transparent huge pages][transhuge], i.e. whether it points at a huge physical
page rather than an ordinary PTE page, under the transparent huge pages scheme.

This checks that the `_PAGE_PSE` flag is set and the `_PAGE_DEVMAP` flag is
_not_ set.

#### Arguments

* `pmd` - The PMD entry we want to determine is marked huge or not under the
  Transparent Huge Pages scheme.

#### Returns

Truthy (non-zero) if the PMD entry is marked huge under the Transparent Huge
Pages scheme.

---

### pud_large()

`int pud_large(pud_t pud)`

[pud_large()][pud_large] determines whether the specified PUD entry is marked
gigantic, indicating it points at a gigantic physical page (1GiB on x86-64.)

The function tests the `_PAGE_PSE` and `_PAGE_PRESENT` flags to determine
whether the page is marked gigantic and not swapped out/otherwise unavailable,
respectively.

#### Arguments

* `pud` - The PUD entry we want to determine is marked gigantic or not.

#### Returns

Truthy (non-zero) if the PUD entry is marked gigantic.

---

### pmd_large()

`int pmd_large(pmd_t pmd)`

[pmd_large()][pmd_large] determines whether the specified PMD entry is marked
huge (2MiB on x86-64), indicating that it points at a huge physical page.

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
it points at a huge (2MiB in x86-64) physical page rather than a PTE.

#### Arguments

* `pmd` - The PMD entry which we want a copy of with the huge flag set.

#### Returns

A copy of the input PMD entry with the huge flag set.

---

### pte_mkhuge()

`pte_t pte_mkhuge(pte_t pte)`

[pte_mkhuge()][pte_mkhuge] returns a new [pte_t][pte_t] which is identical to
the specified PTE entry except that the `_PAGE_PSE` flag is set, indicating that
the underlying physical page is huge (2MiB in x86-64.)

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

[__flush_tlb_global]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L63
[__pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L363
[canon_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L431
[device-mapper]:https://en.wikipedia.org/wiki/Device_mapper
[flush_tlb_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L280
[hugetlb]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/hugetlbpage.txt
[invpcid_flush_all]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/tlbflush.h#L48
[massage_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L372
[mprotect]:http://man7.org/linux/man-pages/man2/mprotect.2.html
[nx-bit]:https://en.wikipedia.org/wiki/NX_bit
[pgd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L689
[pgd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L264
[pgd_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L694
[pgd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L662
[pgprot_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L250
[pgprot_val]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L362
[pgprotval_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L16
[pmd_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L605
[pmd_clear_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L283
[pmd_clear_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L361
[pmd_devmap]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L190
[pmd_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L121
[pmd_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L342
[pmd_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L62
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
[pmd_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L428
[pmd_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L521
[pmd_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L276
[pmd_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L341
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
[pmd_trans_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L179
[pmd_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L850
[pmd_wrprotect]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L300
[pmd_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L126
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
[pte_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L427
[pte_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L491
[pte_set_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L197
[pte_soft_dirty]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L336
[pte_special]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L151
[pte_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_64_types.h#L18
[pte_write]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L131
[pte_wrprotect]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L221
[pte_young]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L116
[pud_bad]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L650
[pud_flags]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L324
[pud_huge]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/hugetlbpage.c#L68
[pud_large]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L644
[pud_none]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L616
[pud_pgprot]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L429
[pud_present]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L621
[soft-dirty]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/soft-dirty.txt
[split_huge_page]:https://github.com/torvalds/linux/blob/v4.6/include/linux/huge_mm.h#L92
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
