# Process Autopsy

* In this section we'll examine in detail how the kernel allocates memory for a
  simple userland program:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>

/* 10MiB */
#define SIZE (10 << 20)

int main(void)
{
        unsigned char *ptr = mmap(NULL, SIZE, PROT_READ | PROT_WRITE,
                                MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (ptr == MAP_FAILED) {
                perror("mmap");
                exit(1);
        }

        puts("mmap        <hit ret>");
        getchar();

        ptr[0] = 'x';
        ptr[SIZE - 1] = 'y';

        puts("0, SIZE - 1 <hit ret>");
        getchar();

        memset(ptr, 'x', SIZE);

        puts("memset      <hit ret>");
        getchar();

        return EXIT_SUCCESS;
}
```

* This program first allocates 10MiB of memory using [mmap()][mmap] before
  putting some data into the first and last pages of memory of the allocated
  block and finally filling the memory, interleaving these steps with waits on
  user input to allow for analysis between each stage.

* It is structured this way so we can examine how the memory is first assigned
  to the process, then partially and fully faulted in (see the
  [process address space][process] section for more details on faulting in.)

## Binary Execution

* After a fork, the binary is executed via the [execve][execve]
  [system call][execve-syscall] which calls [do_execve()][do_execve] and
  [do_execveat_common()][do_execveat_common] in turn.

* `do_execveat_common()` performs various checks, obtains a file descriptor for
  the binary and populates a [struct linux_binprm][linux_binprm] structure to
  contain details regarding the loading of the binary.

* Memory allocation for the new process is performed by
  [bprm_mm_init()][bprm_mm_init] which allocates its new
  [struct mm_struct][mm_struct] memory descriptor which will describe the
  binary's memory layout. `bprm_mm_init()` then calls
  [__bprm_mm_init()][__bprm_mm_init] in turn to perform further memory
  configuration.

* The memory descriptor allocation is performed by [mm_alloc()][mm_alloc] which
  calls [allocate_mm()][allocate_mm], a simple wrapper around a [slab][slab]
  allocation, zeroes the memory and calls [mm_init()][mm_init] to initialise the
  descriptor.

* `mm_init()` sets descriptor fields to appropriate initial values inherits the
  forked process's memory flags and allocates a new PGD (see the section on
  [page tables][page-tables] for details on page tables in general.)

* The PGD is allocated via [mm_alloc_pgd()][mm_alloc_pgd] which calls
  [pgd_alloc()][pgd_alloc] in turn.

* `pgd_alloc()` calls [_pgd_alloc()][_pgd_alloc] to actually allocate the PGD
  itself which (except in the case of a xen paravirtualised configuration) wraps
  a [slab][slab] allocation. In the non-PAE case (we are only considering x86-64
  in these notes so we can neglect PAE behaviour) [pgd_ctor()][pgd_ctor] clones
  kernel mappings into the PGD via [clone_pgd_range()][clone_pgd_range] which is
  a wrapper around the kernel `memcpy()`, starting from
  [KERNEL_PGD_BOUNDARY][KERNEL_PGD_BOUNDARY] and copying
  [KERNEL_PGD_PTRS][KERNEL_PGD_PTRS] entries.

* Once the [struct mm_struct][mm_struct] is set up,
  [bprm_mm_init()][bprm_mm_init] calls [__bprm_mm_init()][__bprm_mm_init] which
  allocates a stack VMA (see [process address space][process] for more details
  on VMA's) at [STACK_TOP_MAX][STACK_TOP_MAX] which on x86-64 is equal to
  [TASK_SIZE_MAX][TASK_SIZE_MAX], the maximum allowed userland address, at a
  size of 1 page.

* This stack allocation is intended to be an initial configuration to be
  adjusted later.
  
[KERNEL_PGD_BOUNDARY]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L722
[KERNEL_PGD_PTRS]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L723
[STACK_TOP_MAX]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L761
[TASK_SIZE_MAX]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L747
[__bprm_mm_init]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L260
[_pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L319
[allocate_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L566
[bprm_mm_init]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L373
[clone_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L879
[do_execve]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1724
[do_execveat_common]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1580
[execve-syscall]:http://man7.org/linux/man-pages/man2/execve.2.html
[execve]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1806
[linux_binprm]:http://github.com/torvalds/linux/blob/v4.6/include/linux/binfmts.h?v=4.6#L14
[mm_alloc]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L674
[mm_alloc_pgd]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L540
[mm_init]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L598
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mmap]:http://man7.org/linux/man-pages/man2/mmap.2.html
[pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L354
[pgd_ctor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L116

[page-tables]:./page-tables.md
[process]:./process.md
[slab]:./slab.md
