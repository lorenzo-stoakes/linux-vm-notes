# Linux VM Notes

## IMPORTANT

These notes are superceded by my new [linux mm
book](https://github.com/lorenzo-stoakes/mm-book) project where I will be trying
to write a book on the subject based on a recent kernel. All future efforts in
this area will be made there.

## Contents

1. [Fundamentals][fundamentals] - Overview of virtual memory concepts without
   referencing any actual code or making assumptions about prior knowledge.

2. [Page Tables][page-tables] - Discussion about what [page tables][page-table]
   are, how they are used in linux, how they are manually traversed, the TLB and
   finally how page tables are allocated and freed.

3. [Process Address Space][process] - Discussion about virtual memory address
  space for processes - managing VMAs, process data structure
  construction/destruction, file/device-backed memory regions, page faulting,
  demand allocation, COW pages, copying to/from userland.

4. [Transparent Huge Pages][trans-huge-pages] - Discussion about the kernel's
   [transparent huge pages][transhuge] functionality.

5. [Out of Memory Killer][oom] - Discussion about overcommit and how the OOM
   killer selects and kills victims.

6. [Process Autopsy][autopsy] - Examining the memory allocation performed by the
   kernel for a simple userland application.

## Introduction

__NOTE:__ I target [linux 4.6][linux-4.6] and an x86-64 non-[NUMA][numa] system.

This repo contains my notes on the linux 4.6 VM subsystem. I don't make any
claim to their quality or usefulness. They are an ongoing project and as such
are a constant work in progress.

This work first originated from the [notes][linux-gorman] I took from the
excellent [Understanding the Linux Virtual Memory Manager][amazon-gorman] by
[Mel Gorman][gorman] which, while great, targets the 2.4.22 kernel (released in
2003.) The obvious next stage of study was to take notes for a recent kernel,
which is what these notes are!

I am specifically targeting the 4.6 kernel since it was the mainline version at
the time of writing and should remain a sane and stable basis for notes and
hacks for the foreseeable future. I may update to newer kernel versions over
time depending on whether it makes sense to do so (and if I succeed at this
project of course!)

## Hacks

Speaking of hacks `linux-vm-notes`'s sister repo, [linux-vm-hacks][vm-hacks], is
where I'm putting exploratory code and patches relating to my exploration of the
VM subsystem.

## License

These notes are licensed under [Creative Commons BY-NC-SA][license].

[amazon-gorman]:http://www.amazon.co.uk/Understanding-Virtual-Memory-Manager-Perens/dp/0131453483
[gorman]:http://www.csn.ul.ie/~mel/blog/
[license]:http://creativecommons.org/licenses/by-nc-sa/4.0/
[linux-4.6]:https://github.com/torvalds/linux/tree/v4.6
[linux-gorman]:https://github.com/lorenzo-stoakes/linux-gorman-book-notes
[numa]:https://en.wikipedia.org/wiki/Non-uniform_memory_access
[page-table]:https://en.wikipedia.org/wiki/Page_table
[transhuge]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/transhuge.txt
[vm-hacks]:https://github.com/lorenzo-stoakes/linux-vm-hacks

[autopsy]:sections/autopsy.md
[fundamentals]:sections/fundamentals.md
[trans-huge-pages]:sections/trans-huge-pages.md
[oom]:sections/oom.md
[page-tables]:sections/page-tables.md
[process]:sections/process.md

[mm-notes]:https://github.com/lorenzo-stoakes/linux-mm-notes
