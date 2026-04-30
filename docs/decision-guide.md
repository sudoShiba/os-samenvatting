# Decision Guide

When modifying the xv6 kernel, you often have multiple ways to achieve a goal. This guide helps you choose the most idiomatic and efficient approach.

## Memory Allocation

### `kalloc` vs `uvmalloc`
- **Use `kalloc()`** when you need a raw physical page (4096 bytes) for the kernel's use (e.g., to create a new page table level or for a COW copy).
- **Use `uvmalloc()`** when you want to increase the virtual size of a user process's address space. It handles both physical allocation (`kalloc`) and page table mapping (`mappages`).

### `mappages` vs `uvmcopy`
- **Use `mappages()`** to create a specific mapping between a virtual address and a physical frame. Common in `mmap` and vDSO implementation.
- **Use `uvmcopy()`** to duplicate an entire address space. Primary used in `fork()`.

## System Call Arguments

### `argint` vs `argaddr`
- **Use `argint()`** to retrieve a simple integer or file descriptor passed as an argument.
- **Use `argaddr()`** to retrieve a pointer (virtual address) passed from user space. Remember that you cannot dereference this address directly in the kernel; use `copyin` or `copyout`.

## Data Movement

### `copyin` vs `copyout`
- **Use `copyin()`** to read data from a user-supplied virtual address into a kernel buffer.
- **Use `copyout()`** to write data from a kernel buffer back to a user-supplied virtual address.
- **Why?** These functions handle the translation from user virtual addresses to kernel physical addresses and check for valid mappings.
