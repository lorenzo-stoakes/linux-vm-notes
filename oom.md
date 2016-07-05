# Out of Memory Killer

* The Out of Memory Killer (OOM killer) is the last resort available to a linux
  system when it is unable to allocate requested memory. It uses heuristics to
  determine process(es) to kill in order to free sufficient physical memory for
  the system to resume normal operation.

* In order for the OOM to reach the state where the OOM killer needs to be
  invoked, the system needs to have 'overcommit' enabled. An overcommit is an
  allocation which either does not take into account available physical memory
  at all, or only rejects really crazy requests.

* There are 3 possible overcommit modes, specified by the [sysctl][sysctl]
  `vm.overcommit_memory` (see the [overcommit accounting][overcommit-accounting]
  documentation):

  * `0` - `OVERCOMMIT_GUESS` - (Default) - Overcommit, but disallow obviously
    crazy allocations. If the process has `CAP_SYS_ADMIN` capability, then more
    memory can be allocated.

  * `1` - `OVERCOMMIT_ALWAYS` - Always overcommit, no matter what. YOLO.

  * `2` - `OVERCOMMIT_NEVER` - Do not overcommit and limit allocations to
    available swap and a proportion of physical RAM, defined by either
    [sysctl][sysctl] `vm.overcommit_ratio`, which defines the maximum percentage
    to use (defaults to 50%), or [sysctl][sysctl] `vm.overcommit_kbytes` which
    specifies an absolute maximum amount of physical memory to allocate in
    kilobytes.

* It's highly likely your system and most you encounter will use the
  `OVERCOMMIT_GUESS` mode. This makes sense, as under
  [demand paging][demand-paging], allocations are lazily allocated via page
  faults (see the section on [process address space][process]), so requiring
  that there is sufficient physical memory to contain blocks of memory that are
  not yet used or observed in any way is highly wasteful.

* As a real-life example of this, I set `vm.overcommit_memory` to
  `OVERCOMMIT_NEVER` on my live host system and chrome was immediately killed :)

[demand-paging]:https://en.wikipedia.org/wiki/Demand_paging
[overcommit-accounting]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/overcommit-accounting
[sysctl]:https://wiki.archlinux.org/index.php/Sysctl

[process]:./process.md
