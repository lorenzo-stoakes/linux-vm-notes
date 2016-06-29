# VMA Functions

## Contents

### Memory Descriptor

* [allocate_mm()](#allocate_mm) - Allocates a new [struct mm_struct][mm_struct]
  from the slab allocator.
* [copy_mm()](#copy_mm) - Initialises the specified
  [struct task_struct][task_struct] with a copy of the current task's
  [struct mm_struct][mm_struct].

### VMA Helpers

* [vma_is_anonymous()](#vma_is_anonymous) - Determines if the specified VMA is
  anonymous (i.e. does not map a file.)

---

## Function Descriptions

### allocate_mm()

`void *allocate_mm(void)`

__NOTE:__ Macro, inferring function signature.

[allocate_mm()][allocate_mm] simply allocates a new
[struct mm_struct][mm_struct] from the slab allocator.

#### Arguments

N/A

#### Returns

A pointer to the newly allocated [struct mm_struct][mm_struct].

### copy_mm()

`int copy_mm(unsigned long clone_flags, struct task_struct *tsk)`

[copy_mm()][copy_mm] copies the current task's [struct mm_struct][mm_struct] to
the specified [struct task_struct][task_struct], resetting statistics as it does
so.

The bulk of the actual duplication is performed by [dup_mm()][dup_mm] and
[dup_mmap()][dup_mmap] which copy the page tables too.

There are some exceptions:

1. If `clone_flags & CLONE_VM` then the current task's
   [struct mm_struct][mm_struct] is shared rather than copied.

2. If this is a kernel thread, i.e. there is no `current->mm` available, the
   copy is aborted.

#### Arguments

* `clone_flags` - The clone flags associated with the copy, if it's the case
  that `clone_flags & CLONE_VM`, then the [struct mm_struct][mm_struct] is not
  copied and is instead shared.

* `tsk` - The task whose `mm` field we want to assign a copy of the current
  task's [struct mm_struct][mm_struct] to.

#### Returns

0 if successful, error code otherwise.

---

### vma_is_anonymous()

`bool vma_is_anonymous(struct vm_area_struct *vma)`

[vma_is_anonymous()][vma_is_anonymous] determines whether the specified VMA
(i.e. [struct vm_area_struct][vm_area_struct]) is anonymous - an anonymous VMA
is one that does not map a file but rather simply represents an area of memory
unassociated with any file.

The function simply tests whether the VMA's `vm_ops` fields is null - if so, no
operations are specified and so the VMA is anonymous, otherwise it is not.

#### Arguments

* `vma` - The VMA whose anonymity we wish to determine.

#### Returns

`true` if the VMA is anonymous, otherwise `false`.

---

[allocate_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L566
[copy_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L958
[dup_mm]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L923
[dup_mmap]:https://github.com/torvalds/linux/blob/v4.6/kernel/fork.c#L408
[mm_struct]:http://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L390
[task_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L1394
[vm_area_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L294
[vma_is_anonymous]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1352
