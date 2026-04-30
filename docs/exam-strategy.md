# Coding Exam Strategy

During the exam, you will likely be asked to extend xv6 with a specific feature. Most tasks fall into one of these categories.

## 1. The Syscall Pattern (Standard Plumbing)
If you need to add a syscall:
1. `user/user.h` (Prototype)
2. `user/usys.pl` (Stub)
3. `kernel/syscall.h` (Number)
4. `kernel/syscall.c` (Table entry + extern)
5. `kernel/sysproc.c` or `sysfile.c` (Implementation)

## 2. The Page Table Walk Pattern
If you need to check or modify memory:
```c
pte_t *pte = walk(p->pagetable, va, 0);
if(pte == 0 || !(*pte & PTE_V)) return -1;
uint64 pa = PTE2PA(*pte);
```

## 3. The Process Iteration Pattern
If you need to affect other processes (e.g., a "kill all" or "list processes" syscall):
```c
struct proc *p;
for(p = proc; p < &proc[NPROC]; p++){
  acquire(&p->lock);
  if(p->state != UNUSED){
    // Do something with p
  }
  release(&p->lock);
}
```

## 4. The File System Pattern
If you need to work with files:
```c
struct file *f;
struct inode *ip;
if(argfd(0, 0, &f) < 0) return -1; // Get file from FD
ip = f->ip;
ilock(ip);
// ... do something with ip->size, ip->type ...
iunlock(ip);
```

## 5. Important Macros & Functions
- `PGROUNDDOWN(va)`: Get start of page.
- `PGROUNDUP(va)`: Get start of next page.
- `PTE2PA(pte)`: Get physical address from PTE.
- `PA2PTE(pa)`: Convert physical address to PTE format.
- `myproc()`: Get current process.
- `cpuid()`: Get current CPU ID.
