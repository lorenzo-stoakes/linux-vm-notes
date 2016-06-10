# Linux VM Notes

## Contents

* [Functions](funcs.md) - List of VM-related kernel functions and what they do,
  categorised by the part of the VM they relate to.

1. [Fundamentals](fundamentals.md) - Overview of virtual memory concepts without
   referencing any actual code or making assumptions about prior knowledge.

### Incomplete

2. [Page Tables](page-tables.md) - Discussion about what
   [page tables][page-table] are, how they are used in linux, how they fit in
   the 64-bit address space, how they are manually traversed, the TLB and
   finally how page tables are allocated and freed.

### (Virtually-)Empty

3. [Physical Pages](physical.md) - Discussion about how physical pages are
   managed in linux.

4. [Process Address Space](process.md) - Discussion about virtual memory address
   space for processes - managing VMAs, process data structure
   construction/destruction, file/device-backed memory regions, page faulting,
   demand allocation, COW pages, copying to/from userland.

5. [Buddy Allocator](buddy.md) - Discussion about the fundamental underlying
   physical memory allocator.

6. [SLUB Allocator](slub.md) - Discussion about the default object allocator.

7. [Out of Memory Killer](oom.md) - Discussion about the OOM killer.

## Assumptions

* I target [linux 4.6][linux-4.6].

* To begin with I'm going to focus on bog-standard x86-64 UMA
  (i.e. non-[NUMA][numa]) - the notes will, without necessarily saying so,
  assume this is the only system in existence.

* I am stupid and constantly make mistakes. Don't take what I say to be
  correct - check everything.

* Arsenal will win the league at some point in the future.

## Introduction

This repo contains my notes on the linux 4.6 VM subsystem. I don't make any claim
to their quality or usefulness :)

This work first originated from the [notes][linux-gorman] I took from the
excellent [Understanding the Linux Virtual Memory Manager][amazon-gorman] by
[Mel Gorman][gorman] which, while great, target the 2.4.22 kernel which was
released in 2003, somewhat out of date, the obvious next stage of study was to
do something similar for a recent kernel.

I am specifically targeting the 4.6 kernel since it is the (almost-)current
mainline version at the time of writing and should remain a sane and stable
basis for notes and hacks for the foreseeable future.

I may update to newer kernel versions over time depending on whether it makes
sense to do so (and if I succeed at this project of course!)

## Hacks

Talking of hacks, the sister repo to this one - [linux-vm-hacks][vm-hacks] - is
where I'm keeping code and patches intended to help me understand the VM
subsystem.

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
[vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks
