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

* [pgd_offset()](#pgd_offset) - Gets virtual address of virtual address's PGD
  entry.
* [pud_offset()](#pud_offset) - Gets virtual address of virtual address's PUD
  entry.
* [pmd_offset()](#pmd_offset) - Gets virtual address of virtual address's PMD
  entry.

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

### virt_to_phys()

`phys_addr_t virt_to_phys(volatile void *address)`

[virt_to_phys()][virt_to_phys] translate kernel-mapped virtual address `address`
to a physical one.

Wrapper around [__pa()][__pa].

#### Arguments

* `address` - Kernel-mapped virtual address to be translated.

#### Returns

Physical address associated with specified virtual address.

### __va()

`void *__va(phys_addr_t address)`

[__va()][__va] does the heavy lifting for `phys_to_virt()`, see above.

__NOTE:__ Macro, inferring function signature.

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



[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40

[pgd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L714
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[pgd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L252
[pud_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L684
[pud_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L270
[pmd_offset]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L639
[pmd_t]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable_types.h#L291
