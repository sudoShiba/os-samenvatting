# `ln.c`

Dit bestand bevat de implementaties voor `user/ln.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> [Geen commentaar in broncode]: Hoofdprogramma om een harde link te maken tussen twee bestanden via de link-syscall.

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  if(argc != 3){
    fprintf(2, "Usage: ln old new\n");
    exit(1);
  }
  if(link(argv[1], argv[2]) < 0)
    fprintf(2, "link %s %s: failed\n", argv[1], argv[2]);
  exit(0);
}

```

