# Function Guide

## Fundamentals

* [phys_to_virt()][phys_to_virt] / [__va()][__va] - Translate from a physical
  memory address to a kernel mapped virtual one.

* [virt_to_phys()][virt_to_phys] / [__pa()][__pa] - Translate from a kernel
  mapped virtual memory address to a physical one.

[phys_to_virt]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L136
[__va]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L54
[virt_to_phys]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/io.h#L118
[__pa]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/page.h#L40
