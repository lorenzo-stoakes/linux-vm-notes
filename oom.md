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

  * `0` - `OVERCOMMIT_GUESS` - (Default) - Heuristic overcommit - Determines the
    available free memory on the system, looking at _actually_ referenced
    memory, not allocated (due to [demand paging][demand-paging], allocations
    are lazy allocated on use by default), if the request is larger than this,
    the allocation is refused. Processes with the `CAP_SYS_ADMIN` capability are
    able to allocate more than those without it. This is an overcommit mode as
    allocations of reasonable sizes are allowed then not taken into account in
    future allocations unless the memory allocated is actually referenced.

  * `1` - `OVERCOMMIT_ALWAYS` - Always overcommit, no matter what. YOLO.

  * `2` - `OVERCOMMIT_NEVER` - Do not overcommit and limit allocations to
    available swap and a proportion of physical RAM, defined by either
    [sysctl][sysctl] `vm.overcommit_ratio`, which defines the maximum percentage
    to use (defaults to 50%), or [sysctl][sysctl] `vm.overcommit_kbytes` which
    specifies an absolute maximum amount of physical memory to allocate in
    kilobytes. Only one of these options can be set at any one time (changing
    one causes the other to be zeroed.)

* It's highly likely your system and most you encounter will use the
  `OVERCOMMIT_GUESS` mode. This makes sense, as under
  [demand paging][demand-paging], allocations are lazily allocated via page
  faults (see the section on [process address space][process]), so requiring
  that there is sufficient physical memory to contain blocks of memory that are
  not yet used or observed in any way is highly wasteful.

* As a real-life example of this, I set `vm.overcommit_memory` to
  `OVERCOMMIT_NEVER` on my live host system and chrome was immediately killed :)

* Note that all modes still make use of [demand paging][demand-paging] - what
  changes is the accounting of available memory.

## Memory Availability Accounting

* The function which determines whether sufficient memory is available is
  [__vm_enough_memory()][__vm_enough_memory] which is invoked via
  [security_vm_enough_memory_mm()][security_vm_enough_memory_mm].

* This function determines the correct behaviour under each of the 3 overcommit
  modes:

* As expected, `OVERCOMMIT_ALWAYS` is a simple case in which regardless of the
  number of requested pages, the function does not return an error.

* Under `OVERCOMMIT_GUESS`, a calculation is performed to determine available
  free pages, using [global_page_state()][global_page_state]:

  * Roughly, the calculation is `available_pages = free_pages + page_cache_pages +
    swap_pages + reclaimable_slab_pages - tmpfs_pages - reserved_pages -
    admin_reserved_pages`.

  * Note that the `admin_reserved_pages` value here is the [sysctl][sysctl]
    `vm.admin_reserve_kbytes`, which defaults to 8MiB.

  * tmpfs is subtracted to avoid double-counting, as these can only be swapped
    out, which does not change the calculation.

* Finally, under `OVERCOMMIT_NEVER`, [vm_commit_limit()][vm_commit_limit] is
  invoked to determine the total available memory based on either
  `vm.overcommit_ratio` or `vm.overcommit_kbytes` - this simply uses either of
  these to determine maximum available memory then adds available swap pages.

  * Then, as with `OVERCOMMIT_GUESS`, the [sysctl][sysctl]
    `vm.admin_reserve_kbytes` is subtracted from total available memory.

  * In order to prevent a single process from growing out of control, the
    smaller of 1/32th of the total allocated VM of the process and the
    [sysctl][sysctl] `vm.user_reserve_kbytes` is subtracted from available
    memory.

  * Finally if the total amount of memory committed, including the newly
    requested amount of memory, is less than the determined available memory the
    function succeeds. The total amount of committed memory is stored in the
    per-CPU variable [vm_committed_as][vm_committed_as], which can be read via
    [vm_memory_committed()][vm_memory_committed].

[__vm_enough_memory]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L481
[demand-paging]:https://en.wikipedia.org/wiki/Demand_paging
[global_page_state]:https://github.com/torvalds/linux/blob/v4.6/include/linux/vmstat.h#L120
[overcommit-accounting]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/overcommit-accounting
[security_vm_enough_memory_mm]:https://github.com/torvalds/linux/blob/v4.6/security/security.c#L216
[sysctl]:https://wiki.archlinux.org/index.php/Sysctl
[vm_commit_limit]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L431
[vm_committed_as]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L449
[vm_memory_committed]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L459

[process]:./process.md
