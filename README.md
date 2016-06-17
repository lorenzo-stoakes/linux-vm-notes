# Linux VM Notes

__NOTE:__ I target [linux 4.6][linux-4.6] and an x86-64 non-[NUMA][numa] system.

## Contents

1. [Fundamentals](fundamentals.md) - Overview of virtual memory concepts without
   referencing any actual code or making assumptions about prior knowledge.

2. [Page Tables](page-tables.md) - Discussion about what
   [page tables][page-table] are, how they are used in linux, how they fit in
   the 64-bit address space, how they are manually traversed, the TLB and
   finally how page tables are allocated and freed.

### In Progress

* [Process Address Space](process.md) - Discussion about virtual memory address
  space for processes - managing VMAs, process data structure
  construction/destruction, file/device-backed memory regions, page faulting,
  demand allocation, COW pages, copying to/from userland.

### Function Cheat Sheets

These pages list useful VM functions with a reasonably detailed description of
each function, their arguments, return value and a link to the function
declaration within the [github mirror][linux-4.6] of the linux kernel source
code:

* [General](./funcs.md)
* [Page Tables](./page-table-funcs.md)

### Forthcoming

* [Physical Pages](physical.md) - Discussion about how physical pages are
  managed in linux.

* [Buddy Allocator](buddy.md) - Discussion about the fundamental underlying
  physical memory allocator.

* [SLUB Allocator](slub.md) - Discussion about the default object
  allocator.

* [Out of Memory Killer](oom.md) - Discussion about the OOM
  killer.

* [Huge Pages](huge.md) - Discussion about huge pages, including
  [transparent huge pages][transhuge].

## Introduction

This repo contains my notes on the linux 4.6 VM subsystem. I don't make any
claim to their quality or usefulness.

This work first originated from the [notes][linux-gorman] I took from the
excellent [Understanding the Linux Virtual Memory Manager][amazon-gorman] by
[Mel Gorman][gorman] which, while great, targets the 2.4.22 kernel (released in
2003.) The the obvious next stage of study was to take notes for a recent
kernel, which is what these notes are!

I am specifically targeting the 4.6 kernel since it was the mainline version at
the time of writing and should remain a sane and stable basis for notes and
hacks for the foreseeable future. I may update to newer kernel versions over
time depending on whether it makes sense to do so (and if I succeed at this
project of course!)

## Hacks

Speaking of hacks `linux-vm-notes`'s sister repo, [linux-vm-hacks][vm-hacks], is
where I'm putting exploratory code and patches relating to my exploration of the
VM subsystem.

## Utilities

For tools that might be of some actual use, [memutils][memutils] is the place to
be.

Additionally, I've created some [kernel hacking scripts][kernel-scripts] that
some people might find useful for configuring, compiling and running kernels
under qemu amongst other things.

## License

These notes are licensed under [Creative Commons BY-NC-SA][license] - basically
do what you want with them as long as you:

1. Use them in a non-commercial setting.

2. Provide proper attribution.

3. Distribute them under the same license.

Take a look at the official Creative Commons site if you need more details.

[amazon-gorman]:http://www.amazon.co.uk/Understanding-Virtual-Memory-Manager-Perens/dp/0131453483
[gorman]:http://www.csn.ul.ie/~mel/blog/
[kernel-scripts]:https://github.com/lorenzo-stoakes/kernel-scripts
[license]:http://creativecommons.org/licenses/by-nc-sa/4.0/
[linux-4.6]:https://github.com/torvalds/linux/tree/v4.6
[linux-gorman]:https://github.com/lorenzo-stoakes/linux-gorman-book-notes
[memutils]:https://github.com/lorenzo-stoakes/memutils
[numa]:https://en.wikipedia.org/wiki/Non-uniform_memory_access
[page-table]:https://en.wikipedia.org/wiki/Page_table
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
[vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks
