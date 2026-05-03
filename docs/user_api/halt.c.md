# `halt.c`

Dit bestand bevat de implementaties voor `user/halt.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> [Geen commentaar in broncode]: Hoofdprogramma om het systeem te stoppen door de halt-syscall aan te roepen.

```c
#include "user/user.h"

int main()
{
  halt();
}

```

