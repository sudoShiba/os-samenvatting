# `echo.c`

Dit bestand bevat de implementaties voor `user/echo.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> [Geen commentaar in broncode]: Hoofdprogramma voor de echo-utility; drukt de argumenten af naar de standaarduitvoer, gescheiden door spaties.

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  int i;

  for(i = 1; i < argc; i++){
    write(1, argv[i], strlen(argv[i]));
    if(i + 1 < argc){
      write(1, " ", 1);
    } else {
      write(1, "\n", 1);
    }
  }
  exit(0);
}

```

