# Fundamentals

__Certainty Level:__ Moderate, very early in development but these are pretty,
well, fundamental facts and I've checked them reasonably well.

__Architecture:__ These notes assume an x86-64 UMA system unless otherwise
specified.

## Basics

* In this section I go over some fundamental principles without referencing any
  code so we have a basis to work with once we start digging in to the kernel. I
  intentionally don't go into too much detail here.

* This content will likely be duplicated elsewhere when we start digging into
  things further.

* This section intentionally doesn't reference any code to keep the fundamental
  principles conceptual without getting too bogged down in detail. The entire
  rest of the notes are ALL about getting so bogged down in detail your body
  ends up preserved for future generations ;)

### Virtual Memory

* In Linux (and most modern OSes) [virtual memory][virtual-memory] is used to
  simplify the management of, and prevent unauthorised access to, physical
  memory. At heart the system is simple and does what it says on the tin -
  EVERYTHING (including the kernel) uses _virtual_ addresses to locate data in
  memory.

* Mappings are kept between these virtual addresses and their corresponding
  'physical' addresses which determine the location of data in terms of its
  actual 'position' in RAM. These mappings are maintained via a
  [Memory Management Unit (MMU)][mmu] (on modern computers the MMU is part of
  the CPU.)

* Each userland (i.e. non-kernel) process has its own separate set of mappings
  or 'virtual address space' which the kernel switches out on
  [context switch][context-switch]. This prevents unauthorised access to memory
  by simply not mapping memory the process doesn't own (or isn't legitimately
  sharing with another process) - since it's enforced by the CPU it's
  [pretty][rowhammer] damn robust.

### Pages

* If you mapped every single byte of RAM from a virtual address to a physical
  one you'd need more memory than the system could possibly store for just the
  mappings - each physical address would take up space equal to the physical
  word size (i.e. 4 bytes on a 32-bit system since a byte is 8 bits, 8 bytes on
  a 64-bit system), and you'd need an address for every byte. This is obviously
  a non-starter.

* The solution is simple - divide memory into chunks (known as '[page][page]'s
  of memory) and map those instead. A modern [x86-64][x86-64] system can use
  page sizes of 4KiB, 2MiB and 1GiB, though in most cases 4KiB is used.

* However we still have a problem - on a 32-bit system with a page size of 4KiB
  each process will take up 4MiB of space just for mappings (4GiB of possible
  addresses divided by 4KiB multiplied by 4 bytes to store each address) most of
  which will be empty the vast majority of the time.

* On a 64-bit system things are immediately ridiculous as you'd need 32PiB
  (i.e. 33,554,432 GiB) of storage for every process which is a little high. To
  be fair, current [x86-64][x86-64] systems allow access to less than the entire
  address space at once - a maximum of 48 bits or 256TiB, which would require a
  mere 256GiB per process.

### Page Tables

* Before we look at how this is resolved, let's think about what a virtual
  address looks like - it's just a number in the computer like any other and
  therefore represented by a series of bits, 0's or 1's. Knowing this we can
  divide an address into a series of sub-addresses (for sanity reasons we'll
  call these indexes instead :)

* For example, the 32-bit address 00001000000001001001011001111000 can be
  divided into 3 separated indexes - 0000100000 (32), 0001001001 (73) and
  011001111000 (1656). We can therefore view a virtual address as a series of
  indexes squished into one, each of which can be made to be of a sane size -
  10, 10 and 12 bits respectively in our example here.

* Now, if we create a table of mappings to physical addresses that uses _only
  the first index_, it will be of a reasonable length, in our example above with
  a 10-bit index (maximum value 2^10 = 1024) we'd only require 1024 * 4 = 4KiB
  of space to store it.

* Obviously 4KiB can't store all possible mappings, but it lets us divide up
  each possible virtual address into 1024 different entries, each of which can
  \- and this is the really important bit \- store a physical address of an
  entirely separate table in which we can use the 2nd index (e.g. 73 in our
  example above) to look up the next mapping.

* We could theoretically divide the address any way we like into a number of
  different possible partitions, only providing the actual physical address of
  the page of memory the virtual address references in the very last table we
  look up. However since the work is actually done by the CPU, the hardware
  architecture of the CPU specifies how things are divided.

* When we finally get the physical address of the page of memory we're
  interested in, we still need to know the precise location of the memory
  address _within_ the page. This is achieved by using the value of the final
  bits (1656 in the example above) to 'offset' into this physical page. In a
  sense this last index is the only 'real' part of a virtual address as it
  directly relates to a physical location.

* What's _crucial_ here is that a table entry can be empty - we keep a hierarchy
  of tables for each mapping that actually exists, but for the vast, vast
  majority of possible virtual addresses that point to nowhere, no memory is
  used at all. This also makes it easy for the CPU to determine that an address
  is not valid - if at any point a table entry is empty, then the address is not
  valid.

* Looking at our example diagrammatically:

```
0000100000 0001001001 011001111000 ->
    32         73         1656

Index 1 = 32
Index 2 = 73
Offset  = 1656


      1st table
0    -----------
.    /         /
.    \         \
.    /         /
.    |---------|
32   |       --------\
.    |---------|     |           2nd table
.    /         /     \---> 0    -----------
.    \         \           .    /         /
.    /         /           .    \         \
1024 -----------           .    /         /
                           .    |---------|
                           73   |       --------\
                           .    |---------|     |         Physical page
                           .    /         /     \---> 0    -----------
                           .    \         \           .    /         /
                           .    /         /           .    \         \
                           1024 -----------           .    /         /
                                                      .    |---------|
                                                      1656 |  ohai!  |
                                                      .    |---------|
                                                      .    /         /
                                                      .    \         \
                                                      .    /         /
                                                      4096 -----------
```

* If there were no other tables needed by the process then it would only take up
  8KiB of ram for mapping tables, which is quite a saving on 4MiB (and thinking
  about similar configurations for 64-bit the savings are astronomical.)

* These tables are called [page table][page-table]s and the CPU automatically
  walks through these in the hardware every time a virtual address is used, and
  since once you're in the CPU's 'paging mode' EVERY address any process whether
  kernel or userland uses is virtual, meaning every address goes through this
  process (OK I'm lying a little bit here - the CPU tries to 'remember',
  i.e. cache, mappings for fast lookup when it can to avoid this costly process,
  but more on that later.)

### Page Tables in Linux

* In linux there are 4 levels of [page table][page-table]s for ALL
  architectures. Now, clearly some architectures don't actually have 4 levels of
  page tables - the example I gave above, which is a real-world 32-bit x86
  non-[PAE][pae] address, uses 2 - however linux works around this cleverly by
  making the width in bits of the indexes for the 'middle' page table levels be
  equal to 0 for architectures which lack them.

* This means that 'middle' table indexes essentially reference nothing, and the
  compiler is smart enough to optimise out the code that handles these
  tables. Everything 'just works' and generic code can be safely written that
  assumes 4 levels regardless of architecture. Pretty nice right?

* I'll get into more details about page tables and their actual implementation
  in kernel code in a later section.

### Caching

* Looking up entries in a number of tables just to determine the actual location
  of some data in memory is a costly process and if it happened each time an
  address was referenced, the computer would get seriously bogged down.

* To work around this a cache is used, the
  [Translation Lookaside Buffer (TLB)][tlb]. This stores the mappings between
  virtual and physical addresses in some very fast memory local to the CPU,
  keeping a set of mappings for recently accessed memory. Since programs often
  access the same memory over and over again, this results in a significant
  speedup.

* I'll get into more detail about the TLB and what the kernel code does with it
  in a later section.

[virtual-memory]:https://en.wikipedia.org/wiki/Virtual_memory
[mmu]:https://en.wikipedia.org/wiki/Memory_management_unit
[context-switch]:https://en.wikipedia.org/wiki/Context_switch
[rowhammer]:https://en.wikipedia.org/wiki/Row_hammer
[page]:https://en.wikipedia.org/wiki/Page_(computer_memory)
[x86-64]:https://en.wikipedia.org/wiki/X86-64
[page-table]:https://en.wikipedia.org/wiki/Page_table
[pae]:https://en.wikipedia.org/wiki/Physical_Address_Extension
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
