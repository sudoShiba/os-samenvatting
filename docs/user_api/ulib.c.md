# `ulib.c`

Dit bestand bevat de implementaties voor `user/ulib.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `start()`
>  wrapper so that it's OK if main() does not call exit(). 

```c
void
start(int argc, char **argv)
{
  int r;
  extern int main(int argc, char **argv);
  r = main(argc, argv);
  exit(r);
}

```

## `strcpy()`
> Geen specifieke commentaar in de broncode.

```c

char*
strcpy(char *s, const char *t)
{
  char *os;

  os = s;
  while((*s++ = *t++) != 0)
    ;
  return os;
}

```

## `strcmp()`
> Geen specifieke commentaar in de broncode.

```c

int
strcmp(const char *p, const char *q)
{
  while(*p && *p == *q)
    p++, q++;
  return (uchar)*p - (uchar)*q;
}

```

## `strlen()`
> Geen specifieke commentaar in de broncode.

```c

uint
strlen(const char *s)
{
  int n;

  for(n = 0; s[n]; n++)
    ;
  return n;
}

```

## `memset()`
> Geen specifieke commentaar in de broncode.

```c

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

## `strchr()`
> Geen specifieke commentaar in de broncode.

```c

char*
strchr(const char *s, char c)
{
  for(; *s; s++)
    if(*s == c)
      return (char*)s;
  return 0;
}

```

## `fgets()`
> Geen specifieke commentaar in de broncode.

```c

char*
fgets(char *buf, int max, int fd)
{
  int i, cc;
  char c;

  for(i=0; i+1 < max; ){
    cc = read(fd, &c, 1);
    if(cc < 1)
      break;
    buf[i++] = c;
    if(c == '\n' || c == '\r')
      break;
  }
  buf[i] = '\0';
  return buf;
}

```

## `gets()`
> Geen specifieke commentaar in de broncode.

```c

char*
gets(char *buf, int max)
{
  return fgets(buf, max, 0);
}

```

## `stat()`
> Geen specifieke commentaar in de broncode.

```c

int
stat(const char *n, struct stat *st)
{
  int fd;
  int r;

  fd = open(n, O_RDONLY);
  if(fd < 0)
    return -1;
  r = fstat(fd, st);
  close(fd);
  return r;
}

```

## `atoi()`
> Geen specifieke commentaar in de broncode.

```c

int
atoi(const char *s)
{
  int n;

  n = 0;
  while('0' <= *s && *s <= '9')
    n = n*10 + *s++ - '0';
  return n;
}

```

## `memmove()`
> Geen specifieke commentaar in de broncode.

```c

void*
memmove(void *vdst, const void *vsrc, int n)
{
  char *dst;
  const char *src;

  dst = vdst;
  src = vsrc;
  if (src > dst) {
    while(n-- > 0)
      *dst++ = *src++;
  } else {
    dst += n;
    src += n;
    while(n-- > 0)
      *--dst = *--src;
  }
  return vdst;
}

```

## `memcmp()`
> Geen specifieke commentaar in de broncode.

```c

int
memcmp(const void *s1, const void *s2, uint n)
{
  const char *p1 = s1, *p2 = s2;
  while (n-- > 0) {
    if (*p1 != *p2) {
      return *p1 - *p2;
    }
    p1++;
    p2++;
  }
  return 0;
}

```

## `memcpy()`
> Geen specifieke commentaar in de broncode.

```c

void *
memcpy(void *dst, const void *src, uint n)
{
  return memmove(dst, src, n);
}

```

