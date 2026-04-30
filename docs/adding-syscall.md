# Adding a System Call

To add a new system call to xv6, follow these steps:

## 1. User-Space Header
Add the system call prototype to **`user/user.h`**.
```c
int my_syscall(int);
```

## 2. User-Space Stub
Add an entry to **`user/usys.pl`**. This script generates the assembly code that uses the `ecall` instruction to enter the kernel.
```perl
entry("my_syscall");
```

## 3. System Call Number
Define a new system call number in **`kernel/syscall.h`**.
```c
#define SYS_my_syscall 22
```

## 4. Kernel-Space Dispatch
Update **`kernel/syscall.c`** to include the new system call in the dispatch table.
```c
// Add extern declaration
extern uint64 sys_my_syscall(void);

// Add to the syscalls array
[SYS_my_syscall] sys_my_syscall,
```

## 5. Kernel Implementation
Implement the system call logic. Most process-related calls go in **`kernel/sysproc.c`**, and file-related calls go in **`kernel/sysfile.c`**.
```c
uint64
sys_my_syscall(void)
{
  int n;
  argint(0, &n); // Retrieve the first integer argument
  // Your logic here
  return 0;
}
```
