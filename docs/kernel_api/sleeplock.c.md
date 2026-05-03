# `sleeplock.c`

Dit bestand bevat de implementaties voor `kernel/sleeplock.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `initsleeplock()`
> Sleeping locks

```c

#include "types.h"
#include "riscv.h"
#include "defs.h"
#include "param.h"
#include "memlayout.h"
#include "spinlock.h"
#include "proc.h"
#include "sleeplock.h"

void
initsleeplock(struct sleeplock *lk, char *name)
{
  initlock(&lk->lk, "sleep lock");
  lk->name = name;
  lk->locked = 0;
  lk->pid = 0;
}

```

## `acquiresleep()`
> [Geen commentaar in broncode]: Verwerft een sleeplock; als de lock al bezet is, gaat het proces slapen totdat deze vrijkomt.

```c

void
acquiresleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  while (lk->locked) {
    sleep(lk, &lk->lk);
  }
  lk->locked = 1;
  lk->pid = myproc()->pid;
  release(&lk->lk);
}

```

## `releasesleep()`
> [Geen commentaar in broncode]: Laat een sleeplock los en maakt eventuele wachtende processen wakker.

```c

void
releasesleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  lk->locked = 0;
  lk->pid = 0;
  wakeup(lk);
  release(&lk->lk);
}

```

## `holdingsleep()`
> [Geen commentaar in broncode]: Controleert of het huidige proces de sleeplock in bezit heeft.

```c

int
holdingsleep(struct sleeplock *lk)
{
  int r;
  
  acquire(&lk->lk);
  r = lk->locked && (lk->pid == myproc()->pid);
  release(&lk->lk);
  return r;
}

```

