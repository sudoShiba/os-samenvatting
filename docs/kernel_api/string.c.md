# `string.c`

Dit bestand bevat de implementaties voor `kernel/string.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `memset()`
> [Geen commentaar in broncode]: Vult een geheugenblok met een specifieke byte-waarde.

```c
#include "types.h"

void*
memset(void *dst, int c, uint n)
{
  char *cdst = (char *) dst;
  int i;
  for(i = 0; i < n; i++){
    cdst[i] = c;
  }
  return dst;
}

```

## `memcmp()`
> [Geen commentaar in broncode]: Vergelijkt twee geheugenblokken byte voor byte.

```c

int
memcmp(const void *v1, const void *v2, uint n)
{
  const uchar *s1, *s2;

  s1 = v1;
  s2 = v2;
  while(n-- > 0){
    if(*s1 != *s2)
      return *s1 - *s2;
    s1++, s2++;
  }

  return 0;
}

```

## `memmove()`
> [Geen commentaar in broncode]: Kopieert een geheugenblok naar een andere locatie, waarbij overlappende gebieden veilig worden afgehandeld.

```c

void*
memmove(void *dst, const void *src, uint n)
{
  const char *s;
  char *d;

  if(n == 0)
    return dst;
  
  s = src;
  d = dst;
  if(s < d && s + n > d){
    s += n;
    d += n;
    while(n-- > 0)
      *--d = *--s;
  } else
    while(n-- > 0)
      *d++ = *s++;

  return dst;
}

```

## `memcpy()`
> memcpy exists to placate GCC.  Use memmove.

```c
void*
memcpy(void *dst, const void *src, uint n)
{
  return memmove(dst, src, n);
}

```

## `strncmp()`
> [Geen commentaar in broncode]: Vergelijkt twee strings tot een maximaal aantal karakters.

```c

int
strncmp(const char *p, const char *q, uint n)
{
  while(n > 0 && *p && *p == *q)
    n--, p++, q++;
  if(n == 0)
    return 0;
  return (uchar)*p - (uchar)*q;
}

```

## `strncpy()`
> [Geen commentaar in broncode]: Kopieert een string naar een buffer tot een maximaal aantal karakters.

```c

char*
strncpy(char *s, const char *t, int n)
{
  char *os;

  os = s;
  while(n-- > 0 && (*s++ = *t++) != 0)
    ;
  while(n-- > 0)
    *s++ = 0;
  return os;
}

```

## `safestrcpy()`
> Like strncpy but guaranteed to NUL-terminate.

```c
char*
safestrcpy(char *s, const char *t, int n)
{
  char *os;

  os = s;
  if(n <= 0)
    return os;
  while(--n > 0 && (*s++ = *t++) != 0)
    ;
  *s = 0;
  return os;
}

```

## `strlen()`
> [Geen commentaar in broncode]: Berekent de lengte van een nul-beëindigde string.

```c

int
strlen(const char *s)
{
  int n;

  for(n = 0; s[n]; n++)
    ;
  return n;
}

```

