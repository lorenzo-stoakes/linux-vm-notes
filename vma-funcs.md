# VMA Functions

## Contents

* [vma_is_anonymous()](#vma_is_anonymous) - Determines if the specified VMA is
  anonymous (i.e. does not map a file.)

---

## Function Descriptions

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

[vm_area_struct]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm_types.h#L294
[vma_is_anonymous]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mm.h#L1352
