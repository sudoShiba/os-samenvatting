# Common Pitfalls

Debugging a kernel is difficult. Avoid these common mistakes to save hours of troubleshooting.

## 1. Address Alignment
RISC-V hardware and the xv6 kernel expect pages to be aligned to 4096-byte boundaries.
- **Pitfall:** Passing an unaligned address to `mappages` or `uvmunmap`.
- **Solution:** Always use `PGROUNDDOWN(addr)` if you need the start of the page, or check alignment with `(addr % PGSIZE) == 0`.

## 2. Lock Management
Deadlocks are a constant threat in a multi-core kernel.
- **Pitfall:** Acquiring `p->lock` while already holding it, or acquiring locks in a different order than other parts of the kernel.
- **Solution:** Always release locks before returning from a function. Use `holdingsleep()` or check `p->lock.locked` if unsure (though idiomatic code should be clear about lock ownership).

## 3. User vs Kernel Addresses
The kernel and user programs live in different address spaces.
- **Pitfall:** Dereferencing a pointer passed from a user program directly in kernel code.
- **Solution:** Always use `walkaddr`, `copyin`, or `copyout`. User addresses are virtual and mapped through the process's page table; the kernel cannot "see" them directly without translation.

## 4. Reference Counting (Lab 5)
When implementing Copy-on-Write, physical pages are shared.
- **Pitfall:** Freeing a physical page in `kfree` when other processes are still mapping it.
- **Solution:** Implement a robust reference counting array. Increment the count in `fork` (or `uvmcopy`) and decrement it in `kfree`. Only return the page to the free list when the count reaches zero.

## 5. Panic on Failure
- **Pitfall:** Calling `panic()` for errors that are recoverable (e.g., a process passing an invalid address to a syscall).
- **Solution:** Return an error code (usually `-1`) to the user process. `panic()` should only be used for unrecoverable internal kernel errors.
