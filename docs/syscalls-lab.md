# Lab 2: System Calls (Expert Depth)

Modifying the kernel boundary is a classic exam task. Focus on argument retrieval and state management.

## 1. Trace Mechanism (`traceme`)
Tracking syscalls for a specific process.

**Kernel Implementation (`kernel/sysproc.c`):**
```c
uint64 sys_traceme(void) {
  int traceme;
  argint(0, &traceme);
  myproc()->traceme = traceme; // Store state in struct proc
  return 0;
}
```

**Execution in `kernel/syscall.c`:**
```c
void syscall(void) {
  int num;
  struct proc *p = myproc();

  p->numsyscalls++; // Increment syscall counter (for getnumsyscalls)
  num = p->trapframe->a7; // a7 holds syscall number

  trace_syscall(); // Check p->traceme and print info

  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    p->trapframe->a0 = syscalls[num]();
  } else {
    p->trapframe->a0 = -1;
  }
}
```

## 2. Advanced Trace (`tracemefd`)
Redirecting trace output to a file descriptor (usually a pipe).
```c
uint64 sys_tracemefd(void) {
  int tracefd;
  argint(0, &tracefd);
  struct proc *p = myproc();
  // Simple assignment, actual redirection happens in the tracing logic
  if (tracefd >= 0) {
    p->tracefd = tracefd;
  }
  return 0;
}
```

## 3. Shebang Support in `exec`
Executing an interpreter if a file starts with `#!`.

**Logic in `kernel/exec.c`:**
```c
static int parse_shebang(struct inode *ip, char *buf, int bufsize) {
  int n_read = readi(ip, 0, (uint64)buf, 0, bufsize);
  if(n_read <= 2 || buf[0] != '#' || buf[1] != '!')
    return -1;

  int i;
  for(i = 2; i < n_read; ++i) {
    if(buf[i] == 0 || buf[i] == '\n') {
      buf[i] = 0;
      break;
    }
  }
  return (buf[i] == 0) ? i : -1;
}

// Inside kexec():
char buf[MAX_SHEBANG];
int shebang_len = parse_shebang(ip, buf, MAX_SHEBANG);
if(shebang_len > 0) {
  char *argv2[MAX_ARGS];
  argv2[0] = buf + 2; // Interpreter path
  // Shift original argv: argv2[1] = original_path, argv2[2] = original_args...
  iunlockput(ip);
  path = argv2[0];
  argv = argv2;
  // Re-open with interpreter path
  ip = namei(path);
}
```

## 4. Syscall Argument Helpers (`kernel/syscall.c`)
The underlying mechanism to fetch arguments from the trapframe.

```c
static uint64 argraw(int n) {
  struct proc *p = myproc();
  switch (n) {
    case 0: return p->trapframe->a0;
    case 1: return p->trapframe->a1;
    // ... up to a5
  }
}

void argint(int n, int *ip) { *ip = argraw(n); }
void argaddr(int n, uint64 *ip) { *ip = argraw(n); }

int argstr(int n, char *buf, int max) {
  uint64 addr;
  argaddr(n, &addr);
  return fetchstr(addr, buf, max);
}
```
**Exam Tip:** Always check if the user-provided address in `argstr` or `argaddr` is valid by using `fetchstr` or `copyin` which verify page table boundaries.
