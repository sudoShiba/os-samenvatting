# `syscall.c`

Dit bestand bevat de implementaties voor `kernel/syscall.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `fetchaddr()`
> Fetch the uint64 at addr from the current process.

```c
int
fetchaddr(uint64 addr, uint64 *ip)
{
  struct proc *p = myproc();
  if(addr >= p->sz || addr+sizeof(uint64) > p->sz) // both tests needed, in case of overflow
    return -1;
  if(copyin(p->pagetable, (char *)ip, addr, sizeof(*ip)) != 0)
    return -1;
  return 0;
}

```

## `fetchstr()`
> Fetch the nul-terminated string at addr from the current process. Returns length of string, not including nul, or -1 for error.

```c
int
fetchstr(uint64 addr, char *buf, int max)
{
  struct proc *p = myproc();
  if(copyinstr(p->pagetable, buf, addr, max) < 0)
    return -1;
  return strlen(buf);
}

```

## `argraw()`
> Geen specifieke commentaar in de broncode.

```c

static uint64
argraw(int n)
{
  struct proc *p = myproc();
  switch (n) {
  case 0:
    return p->trapframe->a0;
  case 1:
    return p->trapframe->a1;
  case 2:
    return p->trapframe->a2;
  case 3:
    return p->trapframe->a3;
  case 4:
    return p->trapframe->a4;
  case 5:
    return p->trapframe->a5;
  }
  panic("argraw");
  return -1;
}

```

## `argint()`
> Fetch the nth 32-bit system call argument.

```c
void
argint(int n, int *ip)
{
  *ip = argraw(n);
}

```

## `argaddr()`
> Retrieve an argument as a pointer. Doesn't check for legality, since copyin/copyout will do that.

```c
void
argaddr(int n, uint64 *ip)
{
  *ip = argraw(n);
}

```

## `argstr()`
> Fetch the nth word-sized system call argument as a null-terminated string. Copies into buf, at most max. Returns string length if OK (including nul), -1 if error.

```c
int
argstr(int n, char *buf, int max)
{
  uint64 addr;
  argaddr(n, &addr);
  return fetchstr(addr, buf, max);
}

```

## `syscall()`
> Geen specifieke commentaar in de broncode.

```c

void
syscall(void)
{
  int num;
  struct proc *p = myproc();

  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    // Use num to lookup the system call function for num, call it,
    // and store its return value in p->trapframe->a0
    p->trapframe->a0 = syscalls[num]();
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}

```

