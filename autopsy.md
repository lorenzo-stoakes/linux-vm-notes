# Process Autopsy

* In this section we'll examine in detail how the kernel allocates memory for a
  simple userland process ([simple_alloc.c][simple_alloc.c] from the sister
  repository to vm-notes - [linux VM hacks][vm-hacks]):

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define PAGES       1025
#define ADDITIONAL  69
#define STACK_COUNT 7000

static void stack_alloc_recur(int count)
{
	char dummy[1024];

	if (count <= 0)
		return;

	dummy[0] = 'x';

	stack_alloc_recur(count-1);

	dummy[1023] = dummy[0];
}

static void stack_alloc(void)
{
	stack_alloc_recur(STACK_COUNT);
}

static void heap_alloc(void)
{
	long i;
	long page_size = sysconf(_SC_PAGESIZE);
	long bytes = PAGES*page_size + ADDITIONAL;
	unsigned char *buf = malloc(bytes);

	for (i = 0; i < bytes; i++)
		buf[i] = 'x';

	free(buf);
}

int main(void)
{
	heap_alloc();
	stack_alloc();

	return EXIT_SUCCESS;
}
```

[simple_alloc.c]:https://github.com/lorenzo-stoakes/linux-vm-hacks/blob/master/experiments/simple_alloc.c
[vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks
