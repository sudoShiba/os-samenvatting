# `kill.c`

Dit bestand bevat de implementaties voor `user/kill.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> [Geen commentaar in broncode]: Hoofdprogramma om een signaal naar een proces te sturen via de kill-syscall.

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char **argv)
{
  int i;

  if(argc < 2){
    fprintf(2, "usage: kill pid...\n");
    exit(1);
  }
  for(i=1; i<argc; i++)
    kill(atoi(argv[i]));
  exit(0);
}

```

