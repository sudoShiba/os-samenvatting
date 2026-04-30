# `zombie.c`

Dit bestand bevat de implementaties voor `user/zombie.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> Create a zombie process that must be reparented at exit.

```c

#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(void)
{
  if(fork() > 0)
    pause(5);  // Let child exit before parent.
  exit(0);
}

```

