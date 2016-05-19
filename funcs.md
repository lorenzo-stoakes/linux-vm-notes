# VM Functions

## Address Translation

### phys_to_virt()

`void *phys_to_virt(phys_addr_t address)`

[phys_to_virt()][phys_to_virt] translates physical address `address` to a
kernel-mapped virtual one.

#### Arguments

* `address` - Physical address to be translated.

#### Returns

Kernel-mapped virtual address.

### virt_to_phys()

`phys_addr_t virt_to_phys(volatile void *address)`

[virt_to_phys()][virt_to_phys] translate kernel-mapped virtual address `address`
to a physical one.

#### Arguments

* `address` - Kernel-mapped virtual address to be translated.

#### Returns

Physical address associated with specified virtual address.

[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
