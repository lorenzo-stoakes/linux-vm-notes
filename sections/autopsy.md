# Process Autopsy

* In this section we'll examine in detail how the kernel allocates memory for a
  simple userland program ([simple_standalone_mmap.c][simple_standalone_mmap.c]
  from my [allox][allox] project):


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

        puts("<hit ret>");
        getchar();

        ptr[0] = 'x';
        ptr[SIZE - 1] = 'y';

        puts("<hit ret>");
        getchar();

        memset(ptr, 'x', SIZE);

        puts("<hit ret>");
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

[allox]:https://github.com/lorenzo-stoakes/allox
[mmap]:http://man7.org/linux/man-pages/man2/mmap.2.html
[simple_standalone_mmap.c]:https://github.com/lorenzo-stoakes/allox/blob/master/simple_standalone_mmap.c

[process]:./process.md
