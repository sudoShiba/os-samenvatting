# Kernel Helpers

The xv6 kernel provides several utility functions to simplify common tasks.

## Argument Retrieval
These functions are used in system call implementations to safely retrieve arguments passed from user space.
- **`argint(int n, int *ip)`**: Gets the $n$-th integer argument.
- **`argaddr(int n, uint64 *ip)`**: Gets the $n$-th pointer argument (as a `uint64`).
- **`argstr(int n, char *buf, int max)`**: Gets the $n$-th string argument.

## Memory Access
- **`copyin(pagetable_t pagetable, char *dst, uint64 srcva, uint64 len)`**: Copies data from user space to kernel space.
- **`copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)`**: Copies data from kernel space to user space.
- **`walkaddr(pagetable_t pagetable, uint64 va)`**: Translates a user virtual address to a physical address.

## Output
- **`printf(char *fmt, ...)`**: Prints formatted text to the console (kernel only).
- **`panic(char *s)`**: Prints an error message and halts the kernel. Use only for fatal errors.

## Locking
- **`acquire(struct spinlock *lk)`**: Acquires a spinlock.
- **`release(struct spinlock *lk)`**: Releases a spinlock.
- **`sleep(void *chan, struct spinlock *lk)`**: Relinquishes the CPU and sleeps until `wakeup(chan)` is called.
- **`wakeup(void *chan)`**: Wakes up all processes sleeping on `chan`.
