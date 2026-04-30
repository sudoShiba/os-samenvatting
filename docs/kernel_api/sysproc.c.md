# `sysproc.c`

Dit bestand bevat de implementaties voor `kernel/sysproc.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `sys_exit()`
> Geen specifieke commentaar in de broncode.

```c
#include "types.h"
#include "riscv.h"
#include "defs.h"
#include "param.h"
#include "memlayout.h"
#include "spinlock.h"
#include "proc.h"
#include "vm.h"

uint64
sys_exit(void)
{
  int n;
  argint(0, &n);
  kexit(n);
  return 0;  // not reached
}

```

## `sys_getpid()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_getpid(void)
{
  return myproc()->pid;
}

```

## `sys_fork()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_fork(void)
{
  return kfork();
}

```

## `sys_wait()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_wait(void)
{
  uint64 p;
  argaddr(0, &p);
  return kwait(p);
}

```

## `sys_sbrk()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_sbrk(void)
{
  uint64 addr;
  int t;
  int n;

  argint(0, &n);
  argint(1, &t);
  addr = myproc()->sz;

  if(t == SBRK_EAGER || n < 0) {
    if(growproc(n) < 0) {
      return -1;
    }
  } else {
    // Lazily allocate memory for this process: increase its memory
    // size but don't allocate memory. If the processes uses the
    // memory, vmfault() will allocate it.
    if(addr + n < addr)
      return -1;
    if(addr + n > TRAPFRAME)
      return -1;
    myproc()->sz += n;
  }
  return addr;
}

```

## `sys_pause()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_pause(void)
{
  int n;
  uint ticks0;

  argint(0, &n);
  if(n < 0)
    n = 0;
  acquire(&tickslock);
  ticks0 = ticks;
  while(ticks - ticks0 < n){
    if(killed(myproc())){
      release(&tickslock);
      return -1;
    }
    sleep(&ticks, &tickslock);
  }
  release(&tickslock);
  return 0;
}

```

## `sys_kill()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_kill(void)
{
  int pid;

  argint(0, &pid);
  return kkill(pid);
}

```

## `sys_uptime()`
> return how many clock tick interrupts have occurred since start.

```c
uint64
sys_uptime(void)
{
  uint xticks;

  acquire(&tickslock);
  xticks = ticks;
  release(&tickslock);
  return xticks;
}

```

## `sys_vmprintmappings()`
> Geen specifieke commentaar in de broncode.

```c

uint64
sys_vmprintmappings(void)
{
  vmprintmappings(myproc()->pagetable);
  return 0;
}

```

