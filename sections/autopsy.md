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

* The newly created VMA is inserted into the memory descriptor via
  [insert_vm_struct()][insert_vm_struct] which appends it to the address-ordered
  list hanging off the `mmap` field in [struct mm_struct][mm_struct] and the
  [red-black tree][red-black-tree] hanging off `mm_rb`.

* Next (passing over various details we aren't interested in)
  [__bprm_mm_init()][__bprm_mm_init] sets [struct linux_binprm][linux_binprm]'s
  `p` field to the architecture's word size bytes below the end of the newly
  created VMA (8 bytes in x86-64) - this is to provide a 0 word at the top of
  the stack.

* Now [bprm_mm_init()][bprm_mm_init] is done, we return to
  [do_execveat_common()][do_execveat_common] which counts input arguments and
  environment variables, checking that arguments are not longer than permitted
  and storing these counts in the `struct linux_binprm`.

* The next point of interest relating to memory management comes from
  [prepare_binprm()][prepare_binprm] which (amongst other things) pre-populates
  [BINPRM_BUF_SIZE][BINPRM_BUF_SIZE] `== 128` bytes of the input binary into
  `struct linux_binprm`'s `buf` field via [kernel_read()][kernel_read]. This
  will later be used to identify the binary and determine what binary loader to
  use to execute it.

* `do_execveat_common()` next copies the process's filename to the top of the
  stack pointed at by `bprm->p` via
  [copy_strings_kernel()][copy_strings_kernel], sets `bprm->exec` equal to
  `bprm->p` and then copies environment variables and arguments via
  [copy_strings()][copy_strings], resulted in a stack that looks like:

```
  ----------------------
  | 0x0000000000000000 | Empty word
  |--------------------|
  |      ./foo\0       | Filename
  |--------------------| ^ <- bprm->exec
  |    envp[envc-1]    | |
  |--------------------| |
  |    envp[envc-2]    | |
  |- - - - - - - - - - | |
  |        ...         | | Environment Variables
  |- - - - - - - - - - | |
  |      envp[1]       | |
  |--------------------| |
  |      envp[0]       | |
  |--------------------| x
  |    argv[argc-1]    | |
  |--------------------| |
  |    argv[argc-2]    | |
  |- - - - - - - - - - | |
  |        ...         | | Arguments
  |- - - - - - - - - - | |
  |      argv[1]       | |
  |--------------------| |
  |      argv[0]       | |
  |--------------------| v <- bprm->p
```

* Pages are allocated as needed via [get_arg_page()][get_arg_page] which
  allocates pages via [get_user_pages_remote()][get_user_pages_remote] which
  ultimately calls [__get_user_pages()][__get_user_pages] which calls
  [find_extend_vma()][find_extend_vma] in turn which calls
  [expand_stack()][expand_stack] to expand the stack (and consequently
  update the VMA) as needed.

* `get_arg_page()` additionally checks that pages allocated are less than or
  equal to the larger of [ARG_MAX][ARG_MAX] (32 pages) or 1/4 the maximum stack
  size for a process to avoid using too much stack for environment
  variables/arguments.

[ARG_MAX]:https://github.com/torvalds/linux/blob/v4.6/include/uapi/linux/limits.h#L7
[BINPRM_BUF_SIZE]:https://github.com/torvalds/linux/blob/v4.6/include/uapi/linux/binfmts.h#L18
[KERNEL_PGD_BOUNDARY]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L722
[KERNEL_PGD_PTRS]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L723
[STACK_TOP_MAX]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L761
[TASK_SIZE_MAX]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L747
[__bprm_mm_init]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L260
[__get_user_pages]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L516
[_pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L319
[allocate_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L566
[bprm_mm_init]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L373
[clone_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/pgtable.h#L879
[copy_strings]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L465
[copy_strings_kernel]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L556
[do_execve]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1724
[do_execveat_common]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1580
[execve-syscall]:http://man7.org/linux/man-pages/man2/execve.2.html
[execve]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1806
[expand_stack]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2212
[find_extend_vma]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2225
[get_arg_page]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L189
[get_user_pages_remote]:https://github.com/torvalds/linux/blob/v4.6/mm/gup.c#L957
[insert_vm_struct]:https://github.com/torvalds/linux/blob/v4.6/mm/mmap.c#L2769
[kernel_read]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L822
[linux_binprm]:http://github.com/torvalds/linux/blob/v4.6/include/linux/binfmts.h#L14
[mm_alloc]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L674
[mm_alloc_pgd]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L540
[mm_init]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L598
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mmap]:http://man7.org/linux/man-pages/man2/mmap.2.html
[pgd_alloc]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L354
[pgd_ctor]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/pgtable.c#L116
[prepare_binprm]:https://github.com/torvalds/linux/blob/v4.6/fs/exec.c#L1436
[red-black-tree]:https://en.wikipedia.org/wiki/Red%E2%80%93black_tree

[page-tables]:./page-tables.md
[process]:./process.md
[slab]:./slab.md
