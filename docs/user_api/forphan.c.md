# `forphan.c`

Dit bestand bevat de implementaties voor `user/forphan.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> Create an orphaned file and check if test-xv6.py recovers it.

```c

#define BUFSZ 500

char buf[BUFSZ];

int
main(int argc, char **argv)
{
  int fd = 0;
  char *s = argv[0];
  struct stat st;
  char *ff = "file0";
  
  if ((fd = open(ff, O_CREATE|O_WRONLY)) < 0) {
    printf("%s: open failed\n", s);
    exit(1);
  }
  if(fstat(fd, &st) < 0){
    fprintf(2, "%s: cannot stat %s\n", s, "ff");
    exit(1);
  }
  if (unlink(ff) < 0) {
    printf("%s: unlink failed\n", s);
    exit(1);
  }
  if (open(ff, O_RDONLY) != -1) {
    printf("%s: open successed\n", s);
    exit(1);
  }
  printf("wait for kill and reclaim %d\n", st.ino);
  // sit around until killed
  for(;;) pause(1000);
}

```

