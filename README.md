# xv6-virtual-memory
**Null-Pointer Dereference**

## Methodology
---
>The purpose of this patch is to add an exception handling when a program dereferences a null pointer. The second goal is to view the address space of the stack and how it's alloted in virtual memory.

The general method to approaching this problem involves finding where xv6 allocates and initializes a user process stack. From there discover the mechanism to use a page and where to initialize page size in the user address space.

Overview of process memory segments:
>note: remember there are two separate processing spaces in the OS -> user space, and kernel space. 
> Kernel space can only be accessed by the user space through system calls.
> User space is allocated by the kernel and the executing program has direct access to.
> ^^ this is space that is chunked up into segments with different purposes and the space in which to modify.

- stack: data needed for a function call (stack frames)
- heap: dynamic memory allocation where memory can be managed by the process, must be managed in languages like c or automated in languages with garbage collection.
- data: contains initialized global and static variables.
- code: instructions for the compiled program are stored.

## Required Changes to xv6
---

> in the memory management unit `mmu.h`, `PGSIZE` default is 4096 bytes.
 
1. Allocation to user space.stack

2. Alter default page size in `exec.c` -> `sz = PGSIZE`

3. In `vm.c` it is required to initialize the loop to start at the`PGSIZE` (4096 bytes) so that is can skip the first page.
   
4. In `syscall.c` a check must be added to check if the pointer is at 0.
   
5. Finally, in the `Makefile` it can no longer start at 0 and must be at `0x1000`

## Testing
---

The testing scheme requires that a null pointer reference does not crash the system and that the OS traps and kill the process. A simple test can be implemented by intentionally referencing null and then trying to use the pointer in a dereference. The following is an example.

```c
// Assign NULL definition to a pointer
  int* null_pointer = NULL;

  // Attempt to assign something to the dereferenced pointer
  *null_pointer = 200;
```

The about should crash the system if the implementation is not performing to the standard.

Initialize the test with:
```c
$ nulltest
```


SYSTEM TESTS:

- [x] Verify `getpinfo` functionality
- [x] Verify `settickets` functionality
- [x] Verify CLI `st` to set tickets
- [x] Verify CLI `ps` to print process info
- [x] Test scheduler is working
- [x] Test scheduler can change tickets

Figure 1. Testing a null pointer dereference and how the OS behaves.

 ![nullpointer](https://github.com/ztbochanski/xv6-mlfq-lottery-scheduler/raw/main/images/settickets.png)

Pass✅

Figure 2. Process priority level increased and tickets count reset.

 ![priority](https://github.com/ztbochanski/xv6-mlfq-lottery-scheduler/raw/main/images/priority.png)

Pass✅


## Summary
---
