# `cat.c`

Dit bestand bevat de implementaties voor `user/cat.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `cat()`
> [Geen commentaar in broncode]: Leest data uit een bestandsbeschrijvingsnummer en schrijft deze naar de standaarduitvoer (stdout).

```c
#include "kernel/types.h"
#include "kernel/fcntl.h"
#include "user/user.h"

char buf[512];

void
cat(int fd)
{
  int n;

  while((n = read(fd, buf, sizeof(buf))) > 0) {
    if (write(1, buf, n) != n) {
      fprintf(2, "cat: write error\n");
      exit(1);
    }
  }
  if(n < 0){
    fprintf(2, "cat: read error\n");
    exit(1);
  }
}

```

## `main()`
> [Geen commentaar in broncode]: Hoofdprogramma voor de cat-utility; opent bestanden of leest van de standaardinvoer en roept de cat-functie aan.

```c

int
main(int argc, char *argv[])
{
  int fd, i;

  if(argc <= 1){
    cat(0);
    exit(0);
  }

  for(i = 1; i < argc; i++){
    if((fd = open(argv[i], O_RDONLY)) < 0){
      fprintf(2, "cat: cannot open %s\n", argv[i]);
      exit(1);
    }
    cat(fd);
    close(fd);
  }
  exit(0);
}

```

