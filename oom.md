# Out of Memory Killer

* The Out of Memory Killer (OOM killer) is the last resort available to a linux
  system when it is unable to allocate requested memory. It uses heuristics to
  determine a process to kill in order to free sufficient physical memory for
  the system to resume normal operation.

* In order for the OOM to reach the state where the OOM killer needs to be
  invoked, the system needs to have 'overcommit' enabled.

* 'Overcommitting' is where memory allocations are permitted under circumstances
  where the actual available memory might not be able to satisfy the
  request. This is possible because of [demand paging][demand-paging] - memory
  requested by processes isn't necessarily allocated right way, rather memory
  ranges are stored in VMAs (see the section on [process address space][process]
  for more on VMAs), and pages are 'faulted in' as necessary.

* There are 3 possible overcommit modes, specified by the [sysctl][sysctl]
  `vm.overcommit_memory` (see the [overcommit accounting][overcommit-accounting]
  documentation):

  * `0` - `OVERCOMMIT_GUESS` - (Default) - Heuristic overcommit - Determines the
    available free memory on the system, looking at _actually_ referenced
    memory, not allocated, if the request is larger than this, the allocation is
    refused. Processes with the `CAP_SYS_ADMIN` capability are able to allocate
    more than those without it. This is an overcommit mode as allocations of
    reasonable sizes are allowed then not taken into account in future
    allocations unless the memory allocated is actually referenced.

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
  [demand paging][demand-paging], allocations are lazily allocated, so requiring
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

## Out of Memory Killer

### Entry Point

* The out of memory killer code is surprisingly short and clean, contained
  within [mm/oom_kill.c][oom_kill.c]. In fact contained in the comment at the
  top of the file is a pleasing comment:

```c
 *  Since we won't call these routines often (on a well-configured
 *  machine) this file will double as a 'coding guide' and a signpost
 *  for newbie kernel hackers. It features several pointers to major
 *  kernel subsystems and hints as to where to find out what things do.
```

* The core function which is invoked by the kernel to do the killing is
  [out_of_memory()][out_of_memory], which is protected by the
  [oom_lock][oom_lock] mutex.

* This is called by the physical page allocator when an allocation fails in
  [__alloc_pages_may_oom()][__alloc_pages_may_oom].

* It is also called by the OOM killer's own
  [pagefault_out_of_memory()][pagefault_out_of_memory] function, which is
  invoked by the page fault handler's [mm_fault_error()][mm_fault_error] in
  cases where the page fault cannot be satisfied due to insufficient memory.

* If a memory cgroup controller is in place with OOM handling enabled,
  [mem_cgroup_oom_synchronize()][mem_cgroup_oom_synchronize] is invoked to
  handle the OOM, otherwise `pagefault_out_of_memory()` invokes
  [out_of_memory()][out_of_memory] while holding the `oom_lock`.

### out_of_memory()

* [out_of_memory()][out_of_memory] is the core function which performs the
  actual work of the OOM killer.

* Let's take a look at the logic of the function diagrammatically:

```
                              -------------------
                              | out_of_memory() |
                              -------------------
                                       |
                                       v
      ----------------      Yes /------------\
      | Return false |<--------/  OOM killer  \
      ----------------         \   disabled?  /
                                \------------/
                                       | No
                                       v
                          /-------------------------\ Yes
                         / Notifier indicates memory \--------------------------------\
                         \   found at last moment?   /                                |
                          \-------------------------/                                 |
                                       | No                                           |
                                       v                                              |
                              /-----------------\ Yes      /-------------\ Yes        |
                             / Current thread is \------->/ Fatal signal  \--\        |
                             \    in userland?   /        \    pending?   /  |        |
                              \-----------------/          \-------------/   |        |
                                       | No                       | No       |        |
                                       v                          v          |        |
  /---------------\     Yes /---------------------\     No /--------------\  |        |
 / OOM invoked via \<------/         sysctl        \<-----/  Exiting but   \ |        |
 \    a sysrq?     /       \ vm.panic_on_oom != 0? /      \  no coredump?  / |        |
  \---------------/         \---------------------/        \--------------/  |        |
   No |       | Yes                    | No                       | Yes      |        |
      |       |                        |                          v          v        |
      v       \------------------------\            ------------------------------    |
  ---------                            |            | Mark current as OOM victim |----\
  | Oops! |                            |            ------------------------------    |
  ---------                            v                                              |
                      /---------------------------------\                             |
                     /              sysctl               \                            |
                     \ vm.oom_kill_allocating_task != 0? /                            |
                      \---------------------------------/                             |
                        Yes |                     | No                                |
                            |                     |                                   |
                            |                     |                                   |
                     /------/                     \----\                              |
                     |                                 |                              |
                     v                                 v                              |
            -------------------            ------------------------                   |
            | Examine current |            | Choose a bad process |                   |
            |     process     |        /-->|     to kill via      |                   |
            -------------------        |   | select_bad_process() |                   |
                     |                 |   ------------------------                   |
                     v                 |               |                              |
             /--------------\ Yes      |               v                              |
            /   Is global    \---------/     Yes /-----------\                        |
            \  init process? /         |   /----/ Bad process \                       |
             \--------------/          |   |    \    found?   /                       |
                     | No              |   |     \-----------/                        |
                     |                 |   |           | No                           |
                     |                 |   |           v                              |
                     |                 |   |   /---------------\ Yes                  |
                     v                 |   |  / OOM invoked via \---------------------\
                /---------\ Yes        |   |  \    a sysrq?     /                     |
               / Is kernel \-----------/   |   \---------------/                      |
               \  thread?  /           |   |           | No                           |
                \---------/            |   |           |              ---------       |
                     | No              |   |           \------------->| Oops! |       |
                     |                 |   |                          ---------       |
                     |                 |   \---------------\                          |
                     |                 |                   |                          |
                     v                 |                   v                          |
           /------------------\ Yes    |   /--------------------------------\ Yes     |
          /  oom_score_adj ==  \-------/  /  OOM_SCAN_ABORT due to another   \--------\
          \ OOM_SCORE_ADJ_MIN? /          \ process dying w/memory reserves? /        |
           \------------------/            \--------------------------------/         |
                     | No                                  | No                       |
                     |                                     |                          |
                     |                                     |                          |
                     |                                 1st | 2nd                      |
                     |                    /---------------/ \------------\            |
                     |                    |                              |            |
                     |                    v                              v            |
                     |          ----------------------          -------------------   |
                     \--------->|  Kill process via  |          | Sleep for 1s to |   |
                                | oom_kill_process() |          | allow for kill  |   |
                                ----------------------          -------------------   |
                                          |                                           |
                                          v                                           |
                                   ---------------                                    |
                                   | Return true |<------------------------------------
                                   ---------------
```

### select_bad_process()

__NOTE:__ In the below threads are examined. However, as threads are simply
processes that share a memory descriptor in linux, all the threads in a
multi-threaded application will share the same score (though theoretically in
the time between examining different threads the value might change.) Since
killing a thread kills all threads in the multi-threaded, it is irrelevant which
one is selected, and in fact the logic prefers the thread group leader for
clarity.

* [select_bad_process()][select_bad_process] determines which thread to
  kill. The core of the function is a loop where it iterates over all _threads_
  in the system and assesses them for their suitability for the chop via a
  points system.

* The first task within the loop is to call
  [oom_scan_process_thread()][oom_scan_process_thread] for the thread - this
  determines one of [enum oom_scan_t][oom_scan_t]:

```c
enum oom_scan_t {
        OOM_SCAN_OK,            /* scan thread and find its badness */
        OOM_SCAN_CONTINUE,      /* do not consider thread for oom kill */
        OOM_SCAN_ABORT,         /* abort the iteration and return */
        OOM_SCAN_SELECT,        /* always select this thread first */
};
```

* The typical case is for this function to return `OOM_SCAN_OK`, however each of
  the other cases are dealt with appropriately - `OOM_SCAN_CONTINUE` simply
  continues the loop and examines the next thread, `OOM_SCAN_ABORT` causes the
  overall [select_bad_process()][select_bad_process] function to return -1 cast
  to an unsigned long which indicates the OOM killer as a whole should abort and
  finally `OOM_SCAN_SELECT` assigns the current thread as the victim and sets
  its points to `ULONG_MAX` to ensure it can only get pipped to the post by a
  subsequent `OOM_SCAN_SELECT`'d thread.

* Looking at the logic that leads to each outcome:

1. Firstly, [oom_unkillable_task()][oom_unkillable_task] is checked to see
   whether the thread cannot be killed (and thus `OOM_SCAN_CONTINUE` is
   returned.) This returns true if the thread is the __global init process__ or
   is a __kernel thread__. There is a check here for when a memory cgroup
   controller is in place, however the function is invoked with this parameter
   NULL when called via `select_bad_process()`. There is also a NUMA case, but
   I'm ignoring crazy NUMA stuff in these notes for the time being :)

2. Next, [test_tsk_thread_flag()][test_tsk_thread_flag] is called with the
   `TIF_MEMDIE` flag set to see if another thread has been OOM killed and is
   therefore dying and releasing memory. If this is the case and the OOM killer
   _wasn't_ invoked via [sysrq][sysrq], the OOM killer can be aborted and
   `OOM_SCAN_ABORT` is returned.

3. If the task does not have a [struct mm_struct][mm_struct] assigned in the
   `mm` field, indicating that either it is a kernel thread or it is exiting so
   it ought to be skipped. Kernel threads are exempt from the
   [remorseless scythe][scythe] of the OOM killer, hence `OOM_SCAN_CONTINUE` is
   returned in this case.

4. Finally, if the thread is marked as the potential original of an OOM
   (i.e. `signal->oom_flags & OOM_FLAG_ORIGIN`), indicated via
   [oom_task_origin()][oom_task_origin] this thread should be selected
   regardless, so `OOM_SCAN_SELECT` is returned. Note that in this case, if more
   than one thread is marked this way, the last encountered will be the one
   selected.

5. If none of the above apply, `OOM_SCAN_OK` is returned.

* If, as in the usual case, the scan indicates no special action is to be taken,
  then the thread's score is calculated via [oom_badness()][oom_badness]
  (discussed below.) If the points exceed any previously chosen thread's points,
  then the thread is stored as that chosen to die. If the returned score is 0,
  the thread is ignored.

* There is a special case when the points are _equal_ to the previously chosen
  thread - if the previously chosen thread is a thread group leader, then the
  thread under examination is _not_ chosen, otherwise it is. This ensures that
  in the case of a multi-threaded application it's the thread group leader that
  is chosen.

* Finally [select_bad_process()][select_bad_process] returns the points of the
  chosen thread multiplied by 1000 and scaled by the total number of pages of
  RAM in the system (i.e. the total physical RAM in page units) added to the
  total number of pages of swap.

* In NUMA-related cases the total RAM value used to scale the score can be
  different as defined by [constrained_alloc()][constrained_alloc], however
  again we are assuming an UMA system.

### oom_badness()

__NOTE:__ If the below logic results in a returned score of 0, this causes
[select_bad_process()][select_bad_process] to ignore the thread for the purposes
of selecting a victim.

* [oom_badness()][oom_badness] determines the 'points' associated with a
  thread. The unit of the value returned by this function is in pages.

* Note that the final result output to users after this function returns is
  scaled by 1000 and divided by total RAM + swap in the system.

* Firstly the function checks to see if the thread is unkillable via
  [oom_unkillable_task()][oom_unkillable_task], see above for the criteria for
  this. If so, the returned score is 0.

* Since the thread might currently be exiting or otherwise not have a memory
  descriptor available at the `mm` field, each sub-thread of the thread is
  examined to see if a memory descriptor can be fetched. If not, the returned
  score is 0.

* Next the thread's `oom_score_adj` value is read. This value is a tunable
  available in `/proc/<pid>/oom_score_adj` ranging from -1000 to 1000. This
  value is directly added to the (user-visible) 'badness' score, therefore a
  higher value makes it more likely the process will be killed, and a lower
  value less likely.

* If the retrieved `oom_score_adj` is set to -1000, i.e. `OOM_SCORE_ADJ_MIN`,
  then [oom_badness()][oom_badness] returns a score of 0 - this indicates that
  the process is not to be killed

* Note that `/proc/<pid>/oom_adj` is kept around for legacy purposes only and
  is simply mapped to an equivalent `oom_score_adj` value.

* Usefully you can read the score `oom_badness()` would return for a process via
  `/proc/<pid>/oom_score`.

#### Scoring

* Once the special cases above are taken into account, the score is calculated
  as follows:

```c
        /*
         * The baseline for the badness score is the proportion of RAM that each
         * task's rss, pagetable and swap space use.
         */
        points = get_mm_rss(p->mm) + get_mm_counter(p->mm, MM_SWAPENTS) +
                atomic_long_read(&p->mm->nr_ptes) + mm_nr_pmds(p->mm);
```

* The score is equal to the sum of the process's RSS (more on this below), swap
  pages, PTE and PMD page count.

* A process's RSS is its _resident set size_ (here retrieved via
  [get_mm_rss()][get_mm_rss]. This is the amount of memory that a process is
  actually using, i.e. that has been page faulted in (see the section on the
  [process address space][process] for more details on page faulting),
  _including_ shared memory from e.g. shared libraries.

* Since shared memory is counted for each process, the fact it is counted
  multiple times does not skew the results.

* If the process has the `CAP_SYS_ADMIN` capability, its OOM score is discounted
  by 3% - this mirrors the bonus provided to such processes in the case of
  `OVERCOMMIT_GUESS`.

* Finally, the `oom_score_adj` is taken into account by multiplying by total
  pages of RAM and swap and dividing by 1000 before directly adding this to the
  final score. Quoting from the [proc fs][proc-doc] documentation:

>Consequently, it is very simple for userspace to define the amount of memory to
>consider for each task.  Setting a /proc/<pid>/oom_score_adj value of +500, for
>example, is roughly equivalent to allowing the remainder of tasks sharing the
>same system, cpuset, mempolicy, or memory controller resources to use at least
>50% more memory.  A value of -500, on the other hand, would be roughly
>equivalent to discounting 50% of the task's allowed memory from being considered
>as scoring against the task.

* [oom_badness()][oom_badness] always returns a positive value, if the score
  calculated so far is negative, it returns 1 (remember 0 indicates that the
  process should not be considered for death.)

### oom_kill_process()

* Once a victim is chosen, [oom_kill_process()][oom_kill_process] performs the
  actual killing.

* Firstly, it checks whether the selected task is already exiting - if so it
  merely marks it as the OOM victim and returns.

* Next, an __important__ detail - rather than kill the selected process, the OOM
  killer will first attempt to kill one of its first-generation child processes
  which does not share a memory descriptor with the parent (i.e. is not a
  thread.)

* It does this by iterating through each thread of the process and each of its
  children, checking whether it shares its memory descriptor with the parent,
  and if not calculating a score via [oom_badness()][oom_badness] and
  reassigning the victim to the one with the highest score.

* Once a victim is selected, it is first sent `SIGKILL` in order to try to break
  up amicably. In case the break up is going to get messy, since its memory is
  going to be reaped whether it likes it or not, it is marked to die via
  [mark_oom_victim()][mark_oom_victim] which simply sets the `TIF_MEMDIE` thread
  flag. This is used to avoid potential livelocks in the code.

* Next, all processes are examined and checked to see whether they share the
  memory descriptor and are not in the same thread group.

* If the discovered process is either a kernel thread, the global init process
  or has its `oom_score_adj` set to `OOM_SCORE_ADJ_MIN` process reaping is
  disabled - this memory might still be used as the shared process will not be
  killed.

* For each ordinary process that shares the memory descriptor, a SIGKILL is
  sent.

* Finally, the OOM grim reaper is woken up via
  [wake_oom_reaper()][wake_oom_reaper]. This ultimately invokes
  [oom_reap_task()][oom_reap_task] to perform the actual unconditional killing
  of the chosen victim.

### oom_reap_task()

* [oom_reap_task()][oom_reap_task] is a wrapper around
  [__oom_reap_task()][__oom_reap_task] which does the heavy lifting of freeing
  the victim's VMAs (see [process address space][process] for more on VMAs.)

* If the call to `__oom_reap_task()` fails, then it is repeated up to
  `MAX_OOM_REAP_RETRIES` times (hardcoded to 10), with a roughly 1/10th second
  delay between each attempt, before the reaper gives up (if this occurs it is
  reported via `dmesg`.)

### __oom_reap_task()

* [__oom_reap_task()][__oom_reap_task] starts by checking whether the memory
  descriptor has either been removed, or has a `mm_users` count of 0 (see
  [process address space][process] on the `mm_users` field.) In either case, the
  reaper need not take any special measures and simply exits with a positive
  result. Otherwise, we increment the `mm_users` field to make sure we keep a
  reference to it.

* If the function then cannot acquire the `mmap_sem` semaphore associated with
  the memory descriptor, it returns indicating the the reaping has failed
  allowing for retries.

* What follows this is a loop over each of the process's VMAs in which we unmap
  memory where it makes sense to do so. We ensure we flush the [TLB][tlb]
  correctly by placing this process between [tlb_gather_mmu()][tlb_gather_mmu]
  and a [tlb_finish_mmu()][tlb_finish_mmu] calls.

* To avoid additional steps that are risky in an OOM system - if the VMA is
  marked as being used by hugetlb, or is locked (i.e. are marked such as to not
  be placed into the swap), or is shared but not anonymous (i.e. file-backed) -
  then no unmapping is attempted.

* When a VMA is selected, the actual unmapping is performed via
  [unmap_page_range()][unmap_page_range], which removes page table mappings
  along with the VMA's physical pages. The TLB is flushed as necessary
  throughout.

* Once this unmapping is complete, debug output is provided to `dmesg` and the
  memory descriptor semaphore is released.

* The process is then marked as having an `oom_score_adj` of `OOM_SCORE_ADJ_MIN`
  as it makes no sense to try to reap memory from this process again if the OOM
  killer needs to be invoked once again - we've just reaped as much memory as we
  can.

* Finally, the `TIF_MEMDIE` flag is cleared from the task via
  [exit_oom_victim()][exit_oom_victim], and we call [mmput()][mmput] to
  decrement the memory descriptor's `mm_user` count - we've done what we can to
  release memory.

[__alloc_pages_may_oom]:https://github.com/torvalds/linux/blob/v4.6/mm/page_alloc.c#L2831
[__oom_reap_task]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L426
[__vm_enough_memory]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L481
[constrained_alloc]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L213
[demand-paging]:https://en.wikipedia.org/wiki/Demand_paging
[exit_oom_victim]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L609
[get_mm_rss]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1445
[global_page_state]:https://github.com/torvalds/linux/blob/v4.6/include/linux/vmstat.h#L120
[mark_oom_victim]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L590
[mem_cgroup_oom_synchronize]:https://github.com/torvalds/linux/blob/v4.6/mm/memcontrol.c#L1630
[mm_fault_error]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L971
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[mmput]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L705
[oom_badness]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L164
[oom_kill.c]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c
[oom_kill_process]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L677
[oom_lock]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L51
[oom_reap_task]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L508
[oom_scan_process_thread]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L271
[oom_scan_t]:https://github.com/torvalds/linux/blob/v4.6/include/linux/oom.h#L46
[oom_task_origin]:https://github.com/torvalds/linux/blob/v4.6/include/linux/oom.h#L68
[oom_unkillable_task]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L136
[out_of_memory]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L849
[overcommit-accounting]:https://github.com/torvalds/linux/blob/v4.6/Documentation/vm/overcommit-accounting
[pagefault_out_of_memory]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L920
[proc-doc]:https://github.com/torvalds/linux/blob/v4.6/Documentation/filesystems/proc.txt
[scythe]:https://www.youtube.com/watch?v=Ov6lWldus3o
[security_vm_enough_memory_mm]:https://github.com/torvalds/linux/blob/v4.6/security/security.c#L216
[select_bad_process]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L302
[sysctl]:https://wiki.archlinux.org/index.php/Sysctl
[sysrq]:https://en.wikipedia.org/wiki/Magic_SysRq_key
[test_tsk_thread_flag]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L2915
[tlb]:https://en.wikipedia.org/wiki/Translation_lookaside_buffer
[tlb_finish_mmu]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L273
[tlb_gather_mmu]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L219
[unmap_page_range]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L1268
[vm_commit_limit]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L431
[vm_committed_as]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L449
[vm_memory_committed]:https://github.com/torvalds/linux/blob/v4.6/mm/util.c#L459
[wake_oom_reaper]:https://github.com/torvalds/linux/blob/v4.6/mm/oom_kill.c#L548

[process]:./process.md
