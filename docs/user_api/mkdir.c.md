# `mkdir.c`

Dit bestand bevat de implementaties voor `user/mkdir.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> Geen specifieke commentaar in de broncode.

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  int i;

  if(argc < 2){
    fprintf(2, "Usage: mkdir files...\n");
    exit(1);
  }

  for(i = 1; i < argc; i++){
    if(mkdir(argv[i]) < 0){
      fprintf(2, "mkdir: %s failed to create\n", argv[i]);
      break;
    }
  }

  exit(0);
}

```

