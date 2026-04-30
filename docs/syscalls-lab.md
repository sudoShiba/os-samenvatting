# Lab 2: System Calls (Expert Depth)

Modifying the kernel boundary is a classic exam task. Focus on argument retrieval and state management.

## 1. Trace Mechanism (`traceme`)
Tracking syscalls for a specific process.
**Kernel Implementation (`kernel/trace.c`):**
```c
uint64 sys_traceme(void) {
  int traceme;
  argint(0, &traceme);
  myproc()->traceme = traceme; // Store state in struct proc
  return 0;
}

// Called from syscall() in kernel/syscall.c
void trace_syscall() {
  struct proc* p = myproc();
  if (p->traceme) {
    int syscall_num = p->trapframe->a7; // a7 holds syscall number
    printf("[%d] syscall %d\n", p->pid, syscall_num);
  }
}
```

## 2. Advanced Trace (`tracemefd`)
Redirecting trace output to a pipe.
```c
uint64 sys_tracemefd(void) {
  int tracefd;
  argint(0, &tracefd);
  // Validation: must be a valid, writable pipe
  struct proc* p = myproc();
  if (tracefd >= 0 && p->ofile[tracefd] && p->ofile[tracefd]->type == FD_PIPE) {
    p->tracefd = tracefd;
  }
  return 0;
}
```

## 3. Shebang Support in `exec`
Executing an interpreter if a file starts with `#!`.
**Logic in `kernel/exec.c`:**
```c
int parse_shebang(struct inode *ip, char *buf, int max) {
  if(readi(ip, 0, (uint64)buf, 0, 2) == 2 && buf[0] == '#' && buf[1] == '!') {
    // Read the rest of the line to find the interpreter path
    return read_line(ip, buf, max);
  }
  return 0;
}

// Inside kexec():
if(parse_shebang(ip, buf, MAX_SHEBANG) > 0) {
  char *interp_path = buf + 2; // Skip #!
  // 1. Prepare new argv: [interp_path, original_path, ...args]
  // 2. Resolve interp_path using namei()
  // 3. Continue execution with the interpreter
}
```

## 4. Syscall Argument Helpers
Essential for implementing any new syscall.
- **`argint(n, &i)`**: Get the $n$-th integer (from `a0, a1, ...`).
- **`argaddr(n, &u)`**: Get the $n$-th address (pointer).
- **`argstr(n, buf, max)`**: Safely copy a user string into a kernel buffer.
- **`copyout(pagetable, dstva, src, len)`**: Write kernel data back to user space.
