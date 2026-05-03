# `usertests.c`

Dit bestand bevat de implementaties voor `user/usertests.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `truncate3()`
> [Geen commentaar in broncode]: Test of het inkorten (truncating) van een bestand naar een specifieke grootte correct werkt en of de data daarbuiten daadwerkelijk verwijderd is.

```c
void
truncate3(char *s)
{
  int fd;
  enum { SZ = 5 };

  unlink("t3");
  fd = open("t3", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create t3 failed\n", s);
    exit(1);
  }
  if(write(fd, "hello", SZ) != SZ){
    printf("%s: write t3 failed\n", s);
    exit(1);
  }
  close(fd);

  fd = open("t3", O_TRUNC | O_RDWR);
  if(fd < 0){
    printf("%s: open t3 failed\n", s);
    exit(1);
  }
  close(fd);

  fd = open("t3", O_RDONLY);
  if(read(fd, buf, SZ) != 0){
    printf("%s: t3 not truncated\n", s);
    exit(1);
  }
  close(fd);
}

```

## `writetest()`
> [Geen commentaar in broncode]: Een eenvoudige test om te controleren of gegevens correct naar een nieuw bestand kunnen worden geschreven en vervolgens correct kunnen worden teruggelezen.

```c
void
writetest(char *s)
{
  int fd;
  enum { SZ = 100 };

  unlink("wt");
  fd = open("writetest", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create writetest failed\n", s);
    exit(1);
  }
  for(int i = 0; i < SZ; i++)
    buf[i] = i;
  if(write(fd, buf, SZ) != SZ){
    printf("%s: write writetest failed\n", s);
    exit(1);
  }
  close(fd);

  fd = open("writetest", O_RDONLY);
  if(fd < 0){
    printf("%s: open writetest failed\n", s);
    exit(1);
  }
  if(read(fd, buf, SZ) != SZ){
    printf("%s: read writetest failed\n", s);
    exit(1);
  }
  for(int i = 0; i < SZ; i++){
    if(buf[i] != (char)i){
      printf("%s: read writetest wrong data\n", s);
      exit(1);
    }
  }
  close(fd);
}

```

## `writebig()`
> [Geen commentaar in broncode]: Test het schrijven van een grote hoeveelheid data (meerdere blokken) om te verifiëren of het bestandssysteem de opslag van grotere bestanden goed afhandelt.

```c
void
writebig(char *s)
{
  int fd;
  enum { SZ = 1000 };

  unlink("wb");
  fd = open("writebig", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create writebig failed\n", s);
    exit(1);
  }
  for(int i = 0; i < SZ; i++){
    memset(buf, i, BSIZE);
    if(write(fd, buf, BSIZE) != BSIZE){
      printf("%s: write writebig failed\n", s);
      exit(1);
    }
  }
  close(fd);

  fd = open("writebig", O_RDONLY);
  if(fd < 0){
    printf("%s: open writebig failed\n", s);
    exit(1);
  }
  for(int i = 0; i < SZ; i++){
    if(read(fd, buf, BSIZE) != BSIZE){
      printf("%s: read writebig failed\n", s);
      exit(1);
    }
    if(buf[0] != (char)i || buf[BSIZE-1] != (char)i){
      printf("%s: read writebig wrong data\n", s);
      exit(1);
    }
  }
  close(fd);
}

```

## `dirtest()`
> [Geen commentaar in broncode]: Controleert basis directory-functionaliteit zoals het maken van mappen, het aanmaken van bestanden in mappen, en of directory-listings consistent zijn.

```c
void
dirtest(char *s)
{
  int fd;

  unlink("dt");
  if(mkdir("dirtest") != 0){
    printf("%s: mkdir dirtest failed\n", s);
    exit(1);
  }
  fd = open("dirtest/f", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create dirtest/f failed\n", s);
    exit(1);
  }
  close(fd);
  if(chdir("dirtest") != 0){
    printf("%s: chdir dirtest failed\n", s);
    exit(1);
  }
  unlink("f");
  if(chdir("..") != 0){
    printf("%s: chdir .. failed\n", s);
    exit(1);
  }
  if(unlink("dirtest") != 0){
    printf("%s: unlink dirtest failed\n", s);
    exit(1);
  }
}

```

## `exectest()`
> [Geen commentaar in broncode]: Test de `exec` systeemoproep door te proberen een programma uit te voeren en te controleren of het procesbeheer en geheugenherstel na een crash van `exec` werken.

```c
void
exectest(char *s)
{
  int pid;

  pid = fork();
  if(pid < 0){
    printf("%s: fork failed\n", s);
    exit(1);
  }
  if(pid == 0){
    char *args[] = { "echo", "hello", 0 };
    exec("echo", args);
    printf("%s: exec failed\n", s);
    exit(1);
  } else {
    wait(0);
  }
}

```

## `forkforkfork()`
> [Geen commentaar in broncode]: Voert herhaaldelijk `fork` uit om te testen of het systeem een groot aantal processen kan beheren en correct kan opruimen nadat ze klaar zijn.

```c
void
forkforkfork(char *s)
{
  int pid;

  for(int i = 0; i < 10; i++){
    pid = fork();
    if(pid < 0){
      printf("%s: fork failed\n", s);
      exit(1);
    }
    if(pid == 0){
      exit(0);
    } else {
      wait(0);
    }
  }
}

```

## `unlinkread()`
> can I unlink a file and still read it?

```c
void
unlinkread(char *s)
{
  enum { SZ = 5 };
  int fd, fd1;

  fd = open("unlinkread", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create unlinkread failed\n", s);
    exit(1);
  }
  write(fd, "hello", SZ);
  close(fd);

  fd = open("unlinkread", O_RDWR);
  if(fd < 0){
    printf("%s: open unlinkread failed\n", s);
    exit(1);
  }
  if(unlink("unlinkread") != 0){
    printf("%s: unlink unlinkread failed\n", s);
    exit(1);
  }

  fd1 = open("unlinkread", O_CREATE | O_RDWR);
  write(fd1, "yyy", 3);
  close(fd1);

  if(read(fd, buf, sizeof(buf)) != SZ){
    printf("%s: unlinkread read failed", s);
    exit(1);
  }
  if(buf[0] != 'h'){
    printf("%s: unlinkread wrong data\n", s);
    exit(1);
  }
  if(write(fd, buf, 10) != 10){
    printf("%s: unlinkread write failed\n", s);
    exit(1);
  }
  close(fd);
  unlink("unlinkread");
}

```

## `linktest()`
> [Geen commentaar in broncode]: Test het aanmaken van hard links (`link`), het ontkoppelen (`unlink`) en of de data behouden blijft via de nieuwe link. Controleert ook of linken naar niet-bestaande bestanden of naar zichzelf faalt.

```c

void
linktest(char *s)
{
  enum { SZ = 5 };
  int fd;

  unlink("lf1");
  unlink("lf2");

  fd = open("lf1", O_CREATE|O_RDWR);
  if(fd < 0){
    printf("%s: create lf1 failed\n", s);
    exit(1);
  }
  if(write(fd, "hello", SZ) != SZ){
    printf("%s: write lf1 failed\n", s);
    exit(1);
  }
  close(fd);

  if(link("lf1", "lf2") < 0){
    printf("%s: link lf1 lf2 failed\n", s);
    exit(1);
  }
  unlink("lf1");

  if(open("lf1", 0) >= 0){
    printf("%s: unlinked lf1 but it is still there!\n", s);
    exit(1);
  }

  fd = open("lf2", 0);
  if(fd < 0){
    printf("%s: open lf2 failed\n", s);
    exit(1);
  }
  if(read(fd, buf, sizeof(buf)) != SZ){
    printf("%s: read lf2 failed\n", s);
    exit(1);
  }
  close(fd);

  if(link("lf2", "lf2") >= 0){
    printf("%s: link lf2 lf2 succeeded! oops\n", s);
    exit(1);
  }

  unlink("lf2");
  if(link("lf2", "lf1") >= 0){
    printf("%s: link non-existent succeeded! oops\n", s);
    exit(1);
  }

  if(link(".", "lf1") >= 0){
    printf("%s: link . lf1 succeeded! oops\n", s);
    exit(1);
  }
}

```

## `concreate()`
> test concurrent create/link/unlink of the same file

```c
void
concreate(char *s)
{
  enum { N = 40 };
  char file[3];
  int i, pid, n, fd;
  char fa[N];
  struct {
    ushort inum;
    char name[DIRSIZ];
  } de;

  file[0] = 'C';
  file[2] = '\0';
  for(i = 0; i < N; i++){
    file[1] = '0' + i;
    unlink(file);
    pid = fork();
    if(pid && (i % 3) == 1){
      link("C0", file);
    } else if(pid == 0 && (i % 5) == 1){
      link("C0", file);
    } else {
      fd = open(file, O_CREATE | O_RDWR);
      if(fd < 0){
        printf("concreate create %s failed\n", file);
        exit(1);
      }
      close(fd);
    }
    if(pid == 0) {
      exit(0);
    } else {
      int xstatus;
      wait(&xstatus);
      if(xstatus != 0)
        exit(1);
    }
  }

  memset(fa, 0, sizeof(fa));
  fd = open(".", 0);
  n = 0;
  while(read(fd, &de, sizeof(de)) > 0){
    if(de.inum == 0)
      continue;
    if(de.name[0] == 'C' && de.name[2] == '\0'){
      i = de.name[1] - '0';
      if(i < 0 || i >= sizeof(fa)){
        printf("%s: concreate weird file %s\n", s, de.name);
        exit(1);
      }
      if(fa[i]){
        printf("%s: concreate duplicate file %s\n", s, de.name);
        exit(1);
      }
      fa[i] = 1;
      n++;
    }
  }
  close(fd);

  if(n != N){
    printf("%s: concreate not enough files in directory listing\n", s);
    exit(1);
  }

  for(i = 0; i < N; i++){
    file[1] = '0' + i;
    pid = fork();
    if(pid < 0){
      printf("%s: fork failed\n", s);
      exit(1);
    }
    if(((i % 3) == 0 && pid == 0) ||
       ((i % 3) == 1 && pid != 0)){
      close(open(file, 0));
      close(open(file, 0));
      close(open(file, 0));
      close(open(file, 0));
      close(open(file, 0));
      close(open(file, 0));
    } else {
      unlink(file);
      unlink(file);
      unlink(file);
      unlink(file);
      unlink(file);
      unlink(file);
    }
    if(pid == 0)
      exit(0);
    else
      wait(0);
  }
}

```

## `linkunlink()`
> another concurrent link/unlink/create test, to look for deadlocks.

```c
void
linkunlink(char *s)
{
  int pid, i;

  unlink("x");
  pid = fork();
  if(pid < 0){
    printf("%s: fork failed\n", s);
    exit(1);
  }

  unsigned int x = (pid ? 1 : 97);
  for(i = 0; i < 100; i++){
    x = x * 1103515245 + 12345;
    if((x % 3) == 0){
      close(open("x", O_RDWR | O_CREATE));
    } else if((x % 3) == 1){
      link("cat", "x");
    } else {
      unlink("x");
    }
  }

  if(pid)
    wait(0);
  else
    exit(0);
}

```

## `subdir()`
> [Geen commentaar in broncode]: Test het aanmaken van diepe directory-structuren, path resolution met `..`, en of directory-operaties (zoals `unlink` op een niet-lege map) correct falen.

```c


void
subdir(char *s)
{
  int fd, cc;

  unlink("ff");
  if(mkdir("dd") != 0){
    printf("%s: mkdir dd failed\n", s);
    exit(1);
  }

  fd = open("dd/ff", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create dd/ff failed\n", s);
    exit(1);
  }
  write(fd, "ff", 2);
  close(fd);

  if(unlink("dd") >= 0){
    printf("%s: unlink dd (non-empty dir) succeeded!\n", s);
    exit(1);
  }

  if(mkdir("/dd/dd") != 0){
    printf("%s: subdir mkdir dd/dd failed\n", s);
    exit(1);
  }

  fd = open("dd/dd/ff", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: create dd/dd/ff failed\n", s);
    exit(1);
  }
  write(fd, "FF", 2);
  close(fd);

  fd = open("dd/dd/../ff", 0);
  if(fd < 0){
    printf("%s: open dd/dd/../ff failed\n", s);
    exit(1);
  }
  cc = read(fd, buf, sizeof(buf));
  if(cc != 2 || buf[0] != 'f'){
    printf("%s: dd/dd/../ff wrong content\n", s);
    exit(1);
  }
  close(fd);

  if(link("dd/dd/ff", "dd/dd/ffff") != 0){
    printf("%s: link dd/dd/ff dd/dd/ffff failed\n", s);
    exit(1);
  }

  if(unlink("dd/dd/ff") != 0){
    printf("%s: unlink dd/dd/ff failed\n", s);
    exit(1);
  }
  if(open("dd/dd/ff", O_RDONLY) >= 0){
    printf("%s: open (unlinked) dd/dd/ff succeeded\n", s);
    exit(1);
  }

  if(chdir("dd") != 0){
    printf("%s: chdir dd failed\n", s);
    exit(1);
  }
  if(chdir("dd/../../dd") != 0){
    printf("%s: chdir dd/../../dd failed\n", s);
    exit(1);
  }
  if(chdir("dd/../../../dd") != 0){
    printf("%s: chdir dd/../../../dd failed\n", s);
    exit(1);
  }
  if(chdir("./..") != 0){
    printf("%s: chdir ./.. failed\n", s);
    exit(1);
  }

  fd = open("dd/dd/ffff", 0);
  if(fd < 0){
    printf("%s: open dd/dd/ffff failed\n", s);
    exit(1);
  }
  if(read(fd, buf, sizeof(buf)) != 2){
    printf("%s: read dd/dd/ffff wrong len\n", s);
    exit(1);
  }
  close(fd);

  if(open("dd/dd/ff", O_RDONLY) >= 0){
    printf("%s: open (unlinked) dd/dd/ff succeeded!\n", s);
    exit(1);
  }

  if(open("dd/ff/ff", O_CREATE|O_RDWR) >= 0){
    printf("%s: create dd/ff/ff succeeded!\n", s);
    exit(1);
  }
  if(open("dd/xx/ff", O_CREATE|O_RDWR) >= 0){
    printf("%s: create dd/xx/ff succeeded!\n", s);
    exit(1);
  }
  if(open("dd", O_CREATE) >= 0){
    printf("%s: create dd succeeded!\n", s);
    exit(1);
  }
  if(open("dd", O_RDWR) >= 0){
    printf("%s: open dd rdwr succeeded!\n", s);
    exit(1);
  }
  if(open("dd", O_WRONLY) >= 0){
    printf("%s: open dd wronly succeeded!\n", s);
    exit(1);
  }
  if(link("dd/ff/ff", "dd/dd/xx") == 0){
    printf("%s: link dd/ff/ff dd/dd/xx succeeded!\n", s);
    exit(1);
  }
  if(link("dd/xx/ff", "dd/dd/xx") == 0){
    printf("%s: link dd/xx/ff dd/dd/xx succeeded!\n", s);
    exit(1);
  }
  if(link("dd/ff", "dd/dd/ffff") == 0){
    printf("%s: link dd/ff dd/dd/ffff succeeded!\n", s);
    exit(1);
  }
  if(mkdir("dd/ff/ff") == 0){
    printf("%s: mkdir dd/ff/ff succeeded!\n", s);
    exit(1);
  }
  if(mkdir("dd/xx/ff") == 0){
    printf("%s: mkdir dd/xx/ff succeeded!\n", s);
    exit(1);
  }
  if(mkdir("dd/dd/ffff") == 0){
    printf("%s: mkdir dd/dd/ffff succeeded!\n", s);
    exit(1);
  }
  if(unlink("dd/xx/ff") == 0){
    printf("%s: unlink dd/xx/ff succeeded!\n", s);
    exit(1);
  }
  if(unlink("dd/ff/ff") == 0){
    printf("%s: unlink dd/ff/ff succeeded!\n", s);
    exit(1);
  }
  if(chdir("dd/ff") == 0){
    printf("%s: chdir dd/ff succeeded!\n", s);
    exit(1);
  }
  if(chdir("dd/xx") == 0){
    printf("%s: chdir dd/xx succeeded!\n", s);
    exit(1);
  }

  if(unlink("dd/dd/ffff") != 0){
    printf("%s: unlink dd/dd/ff failed\n", s);
    exit(1);
  }
  if(unlink("dd/ff") != 0){
    printf("%s: unlink dd/ff failed\n", s);
    exit(1);
  }
  if(unlink("dd") == 0){
    printf("%s: unlink non-empty dd succeeded!\n", s);
    exit(1);
  }
  if(unlink("dd/dd") < 0){
    printf("%s: unlink dd/dd failed\n", s);
    exit(1);
  }
  if(unlink("dd") < 0){
    printf("%s: unlink dd failed\n", s);
    exit(1);
  }
}

```

## `bigwrite()`
> test writes that are larger than the log.

```c
void
bigwrite(char *s)
{
  int fd, sz;

  unlink("bigwrite");
  for(sz = 499; sz < (MAXOPBLOCKS+2)*BSIZE; sz += 471){
    fd = open("bigwrite", O_CREATE | O_RDWR);
    if(fd < 0){
      printf("%s: cannot create bigwrite\n", s);
      exit(1);
    }
    int i;
    for(i = 0; i < 2; i++){
      int cc = write(fd, buf, sz);
      if(cc != sz){
        printf("%s: write(%d) ret %d\n", s, sz, cc);
        exit(1);
      }
    }
    close(fd);
    unlink("bigwrite");
  }
}

```

## `bigfile()`
> [Geen commentaar in broncode]: Test het schrijven en lezen van een bestand dat groter is dan één blok om indirecte blokken te testen.

```c


void
bigfile(char *s)
{
  enum { N = 20, SZ=600 };
  int fd, i, total, cc;

  unlink("bigfile.dat");
  fd = open("bigfile.dat", O_CREATE | O_RDWR);
  if(fd < 0){
    printf("%s: cannot create bigfile", s);
    exit(1);
  }
  for(i = 0; i < N; i++){
    memset(buf, i, SZ);
    if(write(fd, buf, SZ) != SZ){
      printf("%s: write bigfile failed\n", s);
      exit(1);
    }
  }
  close(fd);

  fd = open("bigfile.dat", 0);
  if(fd < 0){
    printf("%s: cannot open bigfile\n", s);
    exit(1);
  }
  total = 0;
  for(i = 0; ; i++){
    cc = read(fd, buf, SZ/2);
    if(cc < 0){
      printf("%s: read bigfile failed\n", s);
      exit(1);
    }
    if(cc == 0)
      break;
    if(cc != SZ/2){
      printf("%s: short read bigfile\n", s);
      exit(1);
    }
    if(buf[0] != i/2 || buf[SZ/2-1] != i/2){
      printf("%s: read bigfile wrong data\n", s);
      exit(1);
    }
    total += cc;
  }
  close(fd);
  if(total != N*SZ){
    printf("%s: read bigfile wrong total\n", s);
    exit(1);
  }
  unlink("bigfile.dat");
}

```

## `fourteen()`
> [Geen commentaar in broncode]: Test of de kernel correct omgaat met de maximale lengte van bestandsnamen (14 karakters, gedefinieerd door `DIRSIZ`).

```c

void
fourteen(char *s)
{
  int fd;

  // DIRSIZ is 14.

  if(mkdir("12345678901234") != 0){
    printf("%s: mkdir 12345678901234 failed\n", s);
    exit(1);
  }
  if(mkdir("12345678901234/123456789012345") != 0){
    printf("%s: mkdir 12345678901234/123456789012345 failed\n", s);
    exit(1);
  }
  fd = open("123456789012345/123456789012345/123456789012345", O_CREATE);
  if(fd < 0){
    printf("%s: create 123456789012345/123456789012345/123456789012345 failed\n", s);
    exit(1);
  }
  close(fd);
  fd = open("12345678901234/12345678901234/12345678901234", 0);
  if(fd < 0){
    printf("%s: open 12345678901234/12345678901234/12345678901234 failed\n", s);
    exit(1);
  }
  close(fd);

  if(mkdir("12345678901234/12345678901234") == 0){
    printf("%s: mkdir 12345678901234/12345678901234 succeeded!\n", s);
    exit(1);
  }
  if(mkdir("123456789012345/12345678901234") == 0){
    printf("%s: mkdir 12345678901234/123456789012345 succeeded!\n", s);
    exit(1);
  }

  // clean up
  unlink("123456789012345/12345678901234");
  unlink("12345678901234/12345678901234");
  unlink("12345678901234/12345678901234/12345678901234");
  unlink("123456789012345/123456789012345/123456789012345");
  unlink("12345678901234/123456789012345");
  unlink("12345678901234");
}

```

## `rmdot()`
> [Geen commentaar in broncode]: Controleert of het verboden is om de speciale mappen `.` (huidige map) en `..` (ouder-map) te verwijderen via `unlink`.

```c

void
rmdot(char *s)
{
  if(mkdir("dots") != 0){
    printf("%s: mkdir dots failed\n", s);
    exit(1);
  }
  if(chdir("dots") != 0){
    printf("%s: chdir dots failed\n", s);
    exit(1);
  }
  if(unlink(".") == 0){
    printf("%s: rm . worked!\n", s);
    exit(1);
  }
  if(unlink("..") == 0){
    printf("%s: rm .. worked!\n", s);
    exit(1);
  }
  if(chdir("/") != 0){
    printf("%s: chdir / failed\n", s);
    exit(1);
  }
  if(unlink("dots/.") == 0){
    printf("%s: unlink dots/. worked!\n", s);
    exit(1);
  }
  if(unlink("dots/..") == 0){
    printf("%s: unlink dots/.. worked!\n", s);
    exit(1);
  }
  if(unlink("dots") != 0){
    printf("%s: unlink dots failed!\n", s);
    exit(1);
  }
}

```

## `dirfile()`
> [Geen commentaar in broncode]: Test de scheiding tussen bestanden en mappen; controleert of je niet kunt `chdir` naar een bestand en of mappen niet geopend kunnen worden met schrijfrechten.

```c

void
dirfile(char *s)
{
  int fd;

  fd = open("dirfile", O_CREATE);
  if(fd < 0){
    printf("%s: create dirfile failed\n", s);
    exit(1);
  }
  close(fd);
  if(chdir("dirfile") == 0){
    printf("%s: chdir dirfile succeeded!\n", s);
    exit(1);
  }
  fd = open("dirfile/xx", 0);
  if(fd >= 0){
    printf("%s: create dirfile/xx succeeded!\n", s);
    exit(1);
  }
  fd = open("dirfile/xx", O_CREATE);
  if(fd >= 0){
    printf("%s: create dirfile/xx succeeded!\n", s);
    exit(1);
  }
  if(mkdir("dirfile/xx") == 0){
    printf("%s: mkdir dirfile/xx succeeded!\n", s);
    exit(1);
  }
  if(unlink("dirfile/xx") == 0){
    printf("%s: unlink dirfile/xx succeeded!\n", s);
    exit(1);
  }
  if(link("README", "dirfile/xx") == 0){
    printf("%s: link to dirfile/xx succeeded!\n", s);
    exit(1);
  }
  if(unlink("dirfile") != 0){
    printf("%s: unlink dirfile failed!\n", s);
    exit(1);
  }

  fd = open(".", O_RDWR);
  if(fd >= 0){
    printf("%s: open . for writing succeeded!\n", s);
    exit(1);
  }
  }
  fd = open(".", 0);
  if(write(fd, "x", 1) > 0){
    printf("%s: write . succeeded!\n", s);
    exit(1);
  }
  close(fd);
}

```

## `iref()`
> test that iput() is called at the end of _namei(). also tests empty file names.

```c
void
iref(char *s)
{
  int i, fd;

  for(i = 0; i < NINODE + 1; i++){
    if(mkdir("irefd") != 0){
      printf("%s: mkdir irefd failed\n", s);
      exit(1);
    }
    if(chdir("irefd") != 0){
      printf("%s: chdir irefd failed\n", s);
      exit(1);
    }

    mkdir("");
    link("README", "");
    fd = open("", O_CREATE);
    if(fd >= 0)
      close(fd);
    fd = open("xx", O_CREATE);
    if(fd >= 0)
      close(fd);
    unlink("xx");
  }

  // clean up
  for(i = 0; i < NINODE + 1; i++){
    chdir("..");
    unlink("irefd");
  }

  chdir("/");
}

```

## `forktest()`
> test that fork fails gracefully the forktest binary also does this, but it runs out of proc entries first. inside the bigger usertests binary, we run out of memory first.

```c
void
forktest(char *s)
{
  enum{ N = 1000 };
  int n, pid;

  for(n=0; n<N; n++){
    pid = fork();
    if(pid < 0)
      break;
    if(pid == 0)
      exit(0);
  }

  if (n == 0) {
    printf("%s: no fork at all!\n", s);
    exit(1);
  }

  if(n == N){
    printf("%s: fork claimed to work 1000 times!\n", s);
    exit(1);
  }

  for(; n > 0; n--){
    if(wait(0) < 0){
      printf("%s: wait stopped early\n", s);
      exit(1);
    }
  }

  if(wait(0) != -1){
    printf("%s: wait got too many\n", s);
    exit(1);
  }
}

```

## `sbrkbasic()`
> [Geen commentaar in broncode]: Basis-test voor de `sbrk` systeemoproep; controleert of geheugenallocatie boven de fysieke limiet faalt en of kleine allocaties (minder dan een pagina) correct werken.

```c

void
sbrkbasic(char *s)
{
  enum { TOOMUCH=1024*1024*1024};
  int i, pid, xstatus;
  char *c, *a, *b;

  // does sbrk() return the expected failure value?
  pid = fork();
  if(pid < 0){
    printf("fork failed in sbrkbasic\n");
    exit(1);
  }
  if(pid == 0){
    a = sbrk(TOOMUCH);
    if(a == (char*)SBRK_ERROR){
      // it's OK if this fails.
      exit(0);
    }
    
    for(b = a; b < a+TOOMUCH; b += PGSIZE){
      *b = 99;
    }
    
    // we should not get here! either sbrk(TOOMUCH)
    // should have failed, or (with lazy allocation)
    // a pagefault should have killed this process.
    exit(1);
  }

  wait(&xstatus);
  if(xstatus == 1){
    printf("%s: too much memory allocated!\n", s);
    exit(1);
  }

  // can one sbrk() less than a page?
  a = sbrk(0);
  for(i = 0; i < 5000; i++){
    b = sbrk(1);
    if(b != a){
      printf("%s: sbrk test failed %d %p %p\n", s, i, a, b);
      exit(1);
    }
    *b = 1;
    a = b + 1;
  }
  pid = fork();
  if(pid < 0){
    printf("%s: sbrk test fork failed\n", s);
    exit(1);
  }
  c = sbrk(1);
  c = sbrk(1);
  if(c != a + 1){
    printf("%s: sbrk test failed post-fork\n", s);
    exit(1);
  }
  if(pid == 0)
    exit(0);
  wait(&xstatus);
  exit(xstatus);
}

```

## `sbrkmuch()`
> [Geen commentaar in broncode]: Test het reserveren van een grote hoeveelheid virtueel geheugen en controleert of deallocatie (negatieve `sbrk`) de pagina's daadwerkelijk vrijgeeft.

```c

void
sbrkmuch(char *s)
{
  enum { BIG=100*1024*1024 };
  char *c, *oldbrk, *a, *lastaddr, *p;
  uint64 amt;

  oldbrk = sbrk(0);

  // can one grow address space to something big?
  a = sbrk(0);
  amt = BIG - (uint64)a;
  p = sbrk(amt);
  if (p != a) {
    printf("%s: sbrk test failed to grow big address space; enough phys mem?\n", s);
    exit(1);
  }

  lastaddr = (char*) (BIG-1);
  *lastaddr = 99;

  // can one de-allocate?
  a = sbrk(0);
  c = sbrk(-PGSIZE);
  if(c == (char*)SBRK_ERROR){
    printf("%s: sbrk could not deallocate\n", s);
    exit(1);
  }
  c = sbrk(0);
  if(c != a - PGSIZE){
    printf("%s: sbrk deallocation produced wrong address, a %p c %p\n", s, a, c);
    exit(1);
  }

  // can one re-allocate that page?
  a = sbrk(0);
  c = sbrk(PGSIZE);
  if(c != a || sbrk(0) != a + PGSIZE){
    printf("%s: sbrk re-allocation failed, a %p c %p\n", s, a, c);
    exit(1);
  }
  if(*lastaddr == 99){
    // should be zero
    printf("%s: sbrk de-allocation didn't really deallocate\n", s);
    exit(1);
  }

  a = sbrk(0);
  c = sbrk(-((char*)sbrk(0) - oldbrk));
  if(c != a){
    printf("%s: sbrk downsize failed, a %p c %p\n", s, a, c);
    exit(1);
  }
}

```

## `kernmem()`
> can we read the kernel's memory?

```c
void
kernmem(char *s)
{
  char *a;
  int pid;

  for(a = (char*)(KERNBASE); a < (char*) (KERNBASE+2000000); a += 50000){
    pid = fork();
    if(pid < 0){
      printf("%s: fork failed\n", s);
      exit(1);
    }
    if(pid == 0){
      printf("%s: oops could read %p = %x\n", s, a, *a);
      exit(1);
    }
    int xstatus;
    wait(&xstatus);
    if(xstatus != -1)  // did kernel kill child?
      exit(1);
  }
}

```

## `MAXVAplus()`
> user code should not be able to write to addresses above MAXVA.

```c
void
MAXVAplus(char *s)
{
  volatile uint64 a = MAXVA;
  for( ; a != 0; a <<= 1){
    int pid;
    pid = fork();
    if(pid < 0){
      printf("%s: fork failed\n", s);
      exit(1);
    }
    if(pid == 0){
      *(char*)a = 99;
      printf("%s: oops wrote %p\n", s, (void*)a);
      exit(1);
    }
    int xstatus;
    wait(&xstatus);
    if(xstatus != -1)  // did kernel kill child?
      exit(1);
  }
}

```

## `sbrkfail()`
> if we run the system out of memory, does it clean up the last failed allocation?

```c
void
sbrkfail(char *s)
{
  enum { BIG=100*1024*1024 };
  int i, xstatus;
  int fds[2];
  char scratch;
  char *c, *a;
  int pids[10];
  int pid;
  int failed;

  failed = 0;
  if(pipe(fds) != 0){
    printf("%s: pipe() failed\n", s);
    exit(1);
  }
  for(i = 0; i < sizeof(pids)/sizeof(pids[0]); i++){
    if((pids[i] = fork()) == 0){
      // allocate a lot of memory
      if (sbrk(BIG - (uint64)sbrk(0)) ==  (char*)SBRK_ERROR)
        write(fds[1], "0", 1);
      else
        write(fds[1], "1", 1);
      // sit around until killed
      for(;;) pause(1000);
    }
    if(pids[i] != -1) {
      read(fds[0], &scratch, 1);
      if(scratch == '0')
        failed = 1;
    }
  }
  if(!failed) {
    printf("%s: no allocation failed; allocate more?\n", s);
  }
  
  // if those failed allocations freed up the pages they did allocate,
  // we'll be able to allocate here
  c = sbrk(PGSIZE);
  for(i = 0; i < sizeof(pids)/sizeof(pids[0]); i++){
    if(pids[i] == -1)
      continue;
    kill(pids[i]);
    wait(0);
  }
  if(c == (char*)SBRK_ERROR){
    printf("%s: failed sbrk leaked memory\n", s);
    exit(1);
  }

  // test running fork with the above allocated page 
  pid = fork();
  if(pid < 0){
    printf("%s: fork failed\n", s);
    exit(1);
  }
  if(pid == 0){
    // allocate a lot of memory. this should produce an error
    a = sbrk(10*BIG);
    if(a == (char*)SBRK_ERROR){
      exit(0);
    }   
    printf("%s: allocate a lot of memory succeeded %d\n", s, 10*BIG);
    exit(1);
  }
  wait(&xstatus);
  if(xstatus != 0)
    exit(1);
}

```

## `sbrkarg()`
> test reads/writes from/to allocated memory

```c
void
sbrkarg(char *s)
{
  char *a;
  int fd, n;

  a = sbrk(PGSIZE);
  fd = open("sbrk", O_CREATE|O_WRONLY);
  unlink("sbrk");
  if(fd < 0)  {
    printf("%s: open sbrk failed\n", s);
    exit(1);
  }
  if ((n = write(fd, a, PGSIZE)) < 0) {
    printf("%s: write sbrk failed\n", s);
    exit(1);
  }
  close(fd);

  // test writes to allocated memory
  a = sbrk(PGSIZE);
  if(pipe((int *) a) != 0){
    printf("%s: pipe() failed\n", s);
    exit(1);
  } 
}

```

## `validatetest()`
> [Geen commentaar in broncode]: Probeert de kernel te laten crashen door ongeldige pointers (bijv. buiten het user-bereik) door te geven aan systeemoproepen zoals `link`.

```c

void
validatetest(char *s)
{
  int hi;
  uint64 p;

  hi = 1100*1024;
  for(p = 0; p <= (uint)hi; p += PGSIZE){
    // try to crash the kernel by passing in a bad string pointer
    if(link("nosuchfile", (char*)p) != -1){
      printf("%s: link should not succeed\n", s);
      exit(1);
    }
  }
}

```

## `bsstest()`
> does uninitialized data start out zero?

```c
char uninit[10000];
void
bsstest(char *s)
{
  int i;

  for(i = 0; i < sizeof(uninit); i++){
    if(uninit[i] != '\0'){
      printf("%s: bss test failed\n", s);
      exit(1);
    }
  }
}

```

## `bigargtest()`
> does exec return an error if the arguments are larger than a page? or does it write below the stack and wreck the instructions/data?

```c
void
bigargtest(char *s)
{
  int pid, fd, xstatus;

  unlink("bigarg-ok");
  pid = fork();
  if(pid == 0){
    static char *args[MAXARG];
    int i;
    char big[400];
    memset(big, ' ', sizeof(big));
    big[sizeof(big)-1] = '\0';
    for(i = 0; i < MAXARG-1; i++)
      args[i] = big;
    args[MAXARG-1] = 0;
    // this exec() should fail (and return) because the
    // arguments are too large.
    exec("echo", args);
    fd = open("bigarg-ok", O_CREATE);
    close(fd);
    exit(0);
  } else if(pid < 0){
    printf("%s: bigargtest: fork failed\n", s);
    exit(1);
  }
  
  wait(&xstatus);
  if(xstatus != 0)
    exit(xstatus);
  fd = open("bigarg-ok", 0);
  if(fd < 0){
    printf("%s: bigarg test failed!\n", s);
    exit(1);
  }
  close(fd);
}

```

## `fsfull()`
> what happens when the file system runs out of blocks? answer: balloc panics, so this test is not useful.

```c
void
fsfull()
{
  int nfiles;
  int fsblocks = 0;

  printf("fsfull test\n");

  for(nfiles = 0; ; nfiles++){
    char name[64];
    name[0] = 'f';
    name[1] = '0' + nfiles / 1000;
    name[2] = '0' + (nfiles % 1000) / 100;
    name[3] = '0' + (nfiles % 100) / 10;
    name[4] = '0' + (nfiles % 10);
    name[5] = '\0';
    printf("writing %s\n", name);
    int fd = open(name, O_CREATE|O_RDWR);
    if(fd < 0){
      printf("open %s failed\n", name);
      break;
    }
    int total = 0;
    while(1){
      int cc = write(fd, buf, BSIZE);
      if(cc < BSIZE)
        break;
      total += cc;
      fsblocks++;
    }
    printf("wrote %d bytes\n", total);
    close(fd);
    if(total == 0)
      break;
  }

  while(nfiles >= 0){
    char name[64];
    name[0] = 'f';
    name[1] = '0' + nfiles / 1000;
    name[2] = '0' + (nfiles % 1000) / 100;
    name[3] = '0' + (nfiles % 100) / 10;
    name[4] = '0' + (nfiles % 10);
    name[5] = '\0';
    unlink(name);
    nfiles--;
  }

  printf("fsfull test finished\n");
}

```

## `argptest()`
> [Geen commentaar in broncode]: Test of systeemoproepen zoals `read` correct omgaan met pointers die net op de grens van het toegewezen user-geheugen liggen.

```c

void argptest(char *s)
{
  int fd;
  fd = open("init", O_RDONLY);
  if (fd < 0) {
    printf("%s: open failed\n", s);
    exit(1);
  }
  read(fd, sbrk(0) - 1, -1);
  close(fd);
}

```

## `stacktest()`
> check that there's an invalid page beneath the user stack, to catch stack overflow.

```c
void
stacktest(char *s)
{
  int pid;
  int xstatus;
  
  pid = fork();
  if(pid == 0) {
    char *sp = (char *) r_sp();
    sp -= USERSTACK*PGSIZE;
    // the *sp should cause a trap.
    printf("%s: stacktest: read below stack %d\n", s, *sp);
    exit(1);
  } else if(pid < 0){
    printf("%s: fork failed\n", s);
    exit(1);
  }
  wait(&xstatus);
  if(xstatus == -1)  // kernel killed child?
    exit(0);
  else
    exit(xstatus);
}

```

## `nowrite()`
> check that writes to a few forbidden addresses cause a fault, e.g. process's text and TRAMPOLINE.

```c
void
nowrite(char *s)
{
  int pid;
  int xstatus;
  uint64 addrs[] = { 0, 0x80000000LL, 0x3fffffe000, 0x3ffffff000, 0x4000000000,
                     0xffffffffffffffff };
  
  for(int ai = 0; ai < sizeof(addrs)/sizeof(addrs[0]); ai++){
    pid = fork();
    if(pid == 0) {
      volatile int *addr = (int *) addrs[ai];
      *addr = 10;
      printf("%s: write to %p did not fail!\n", s, addr);
      exit(0);
    } else if(pid < 0){
      printf("%s: fork failed\n", s);
      exit(1);
    }
    wait(&xstatus);
    if(xstatus == 0){
      // kernel did not kill child!
      exit(1);
    }
  }
  exit(0);
}

```

## `pgbug()`
> regression test. copyin(), copyout(), and copyinstr() used to cast the virtual page address to uint, which (with certain wild system call arguments) resulted in a kernel page faults.

```c
void *big = (void*) 0xeaeb0b5b00002f5e;
void
pgbug(char *s)
{
  char *argv[1];
  argv[0] = 0;
  exec(big, argv);
  pipe(big);

  exit(0);
}

```

## `sbrkbugs()`
> regression test. does the kernel panic if a process sbrk()s its size to be less than a page, or zero, or reduces the break by an amount too small to cause a page to be freed?

```c
void
sbrkbugs(char *s)
{
  int pid = fork();
  if(pid < 0){
    printf("fork failed\n");
    exit(1);
  }
  if(pid == 0){
    int sz = (uint64) sbrk(0);
    // free all user memory; there used to be a bug that
    // would not adjust p->sz correctly in this case,
    // causing exit() to panic.
    sbrk(-sz);
    // user page fault here.
    exit(0);
  }
  wait(0);

  pid = fork();
  if(pid < 0){
    printf("fork failed\n");
    exit(1);
  }
  if(pid == 0){
    int sz = (uint64) sbrk(0);
    // set the break to somewhere in the very first
    // page; there used to be a bug that would incorrectly
    // free the first page.
    sbrk(-(sz - 3500));
    exit(0);
  }
  wait(0);

  pid = fork();
  if(pid < 0){
    printf("fork failed\n");
    exit(1);
  }
  if(pid == 0){
    // set the break in the middle of a page.
    sbrk((10*PGSIZE + 2048) - (uint64)sbrk(0));

    // reduce the break a bit, but not enough to
    // cause a page to be freed. this used to cause
    // a panic.
    sbrk(-10);

    exit(0);
  }
  wait(0);

  exit(0);
}

```

## `sbrklast()`
> if process size was somewhat more than a page boundary, and then shrunk to be somewhat less than that page boundary, can the kernel still copyin() from addresses in the last page?

```c
void
sbrklast(char *s)
{
  uint64 top = (uint64) sbrk(0);
  if((top % PGSIZE) != 0)
    sbrk(PGSIZE - (top % PGSIZE));
  sbrk(PGSIZE);
  sbrk(10);
  sbrk(-20);
  top = (uint64) sbrk(0);
  char *p = (char *) (top - 64);
  p[0] = 'x';
  p[1] = '\0';
  int fd = open(p, O_RDWR|O_CREATE);
  write(fd, p, 1);
  close(fd);
  fd = open(p, O_RDWR);
  p[0] = '\0';
  read(fd, p, 1);
  if(p[0] != 'x')
    exit(1);
}

```

## `sbrk8000()`
> does sbrk handle signed int32 wrap-around with negative arguments?

```c
void
sbrk8000(char *s)
{
  sbrk(0x80000004);
  volatile char *top = sbrk(0);
  *(top-1) = *(top-1) + 1;
}

```

## `badarg()`
> regression test. test whether exec() leaks memory if one of the arguments is invalid. the test passes if the kernel doesn't panic.

```c
void
badarg(char *s)
{
  for(int i = 0; i < 50000; i++){
    char *argv[2];
    argv[0] = (char*)0xffffffff;
    argv[1] = 0;
    exec("echo", argv);
  }
  
  exit(0);
}

```

## `lazy_alloc()`
> Touch a page every 64 pages, which with lazy allocation causes one page to be allocated.

```c
void
lazy_alloc(char *s)
{
  char *i, *prev_end, *new_end;
  
  prev_end = sbrklazy(REGION_SZ);
  if (prev_end == (char *) SBRK_ERROR) {
    printf("sbrklazy() failed\n");
    exit(1);
  }
  new_end = prev_end + REGION_SZ;

  for (i = prev_end + PGSIZE; i < new_end; i += 64 * PGSIZE)
    *(char **)i = i;

  for (i = prev_end + PGSIZE; i < new_end; i += 64 * PGSIZE) {
    if (*(char **)i != i) {
      printf("failed to read value from memory\n");
      exit(1);
    }
  }

  exit(0);
}

```

## `lazy_unmap()`
> Touch a page every 64 pages in region, which with lazy allocation causes one page to be allocated. Check that freeing the region frees the allocated pages.

```c
void
lazy_unmap(char *s)
{
  int pid;
  char *i, *prev_end, *new_end;

  prev_end = sbrklazy(REGION_SZ);
  if (prev_end == (char*)SBRK_ERROR) {
    printf("sbrklazy() failed\n");
    exit(1);
  }
  new_end = prev_end + REGION_SZ;

  for (i = prev_end + PGSIZE; i < new_end; i += PGSIZE * PGSIZE)
    *(char **)i = i;

  for (i = prev_end + PGSIZE; i < new_end; i += PGSIZE * PGSIZE) {
    pid = fork();
    if (pid < 0) {
      printf("error forking\n");
      exit(1);
    } else if (pid == 0) {
      sbrklazy(-1L * REGION_SZ);
      *(char **)i = i;
      exit(0);
    } else {
      int status;
      wait(&status);
      if (status == 0) {
        printf("memory not unmapped\n");
        exit(1);
      }
    }
  }

  exit(0);
}

```

## `lazy_copy()`
> [Geen commentaar in broncode]: Test de correcte werking van Copy-on-Write of Lazy Allocation wanneer systeemoproepen (zoals `open`) data proberen te lezen van pagina's die nog niet fysiek gealloceerd zijn.

```c

void
lazy_copy(char *s)
{
  // copyinstr on lazy page
  {
    char *p = sbrk(0);
    sbrklazy(4*PGSIZE);
    open(p + 8192, 0);
  }
  
  {
    void *xx = sbrk(0);
    void *ret = sbrk(-(((uint64) xx)+1));
    if(ret != xx){
      printf("sbrk(sbrk(0)+1) returned %p, not old sz\n", ret);
      exit(1);
    }
  }

  
  // read() and write() to these addresses should fail.
  unsigned long bad[] = {
    0x3fffffc000,
    0x3fffffd000,
    0x3fffffe000,
    0x3ffffff000,
    0x4000000000,
    0x8000000000,
  };
  for(int i = 0; i < sizeof(bad)/sizeof(bad[0]); i++){
    int fd = open("README", 0);
    if(fd < 0) { printf("cannot open README\n"); exit(1); }
    if(read(fd, (char*)bad[i], 512) >= 0) { printf("read succeeded\n");  exit(1); }
    close(fd);
    fd = open("junk", O_CREATE|O_RDWR|O_TRUNC);
    if(fd < 0) { printf("cannot open junk\n"); exit(1); }
    if(write(fd, (char*)bad[i], 512) >= 0) { printf("write succeeded\n"); exit(1); }
    close(fd);
  }

  exit(0);
}

```

## `lazy_sbrk()`
> [Geen commentaar in broncode]: Test of `sbrk` (of `sbrklazy`) correct grote hoeveelheden geheugen kan reserveren tot aan de `MAXVA` grens en of het geheugen correct ge-zero-vuld is.

```c

void
lazy_sbrk(char *s)
{
  // sbrk() takes just int, so take 2^30-sized steps towards MAXVA
  char *p = sbrk(0);
  while ((uint64)p < MAXVA-(1<<30)) {
    p = sbrklazy(1<<30);
    if (p < 0) {
      printf("sbrklazy(%d) returned %p\n", 1<<30, p);
      exit(1);
    }

    p = sbrklazy(0);
  }

  int n = TRAPFRAME-PGSIZE-(uint64)p;

  char *p1 = sbrklazy(n);
  if (p1 < 0 || p1 != p) {
    printf("sbrklazy(%d) returned %p, not expected %p\n", n, p1, p);
    exit(1);
  }

  p = sbrk(PGSIZE);
  if (p < 0 || (uint64)p != TRAPFRAME-PGSIZE) {
    printf("sbrk(%d) returned %p, not expected TRAPFRAME-PGSIZE\n", PGSIZE, p);
    exit(1);
  }

  p[0] = 1;
  if (p[1] != 0) {
    printf("sbrk() returned non-zero-filled memory\n");
    exit(1);
  }

  p = sbrk(1);
  if ((uint64)p != -1) {
    printf("sbrk(1) returned %p, expected error\n", p);
    exit(1);
  }

  p = sbrklazy(1);
  if ((uint64)p != -1) {
    printf("sbrklazy(1) returned %p, expected error\n", p);
    exit(1);
  }

  exit(0);
}

```

## `bigdir()`
>  Section with tests that take a fair bit of time  directory that uses indirect blocks

```c
void
bigdir(char *s)
{
  enum { N = 500 };
  int i, fd;
  char name[10];

  unlink("bd");

  fd = open("bd", O_CREATE);
  if(fd < 0){
    printf("%s: bigdir create failed\n", s);
    exit(1);
  }
  close(fd);

  for(i = 0; i < N; i++){
    name[0] = 'x';
    name[1] = '0' + (i / 64);
    name[2] = '0' + (i % 64);
    name[3] = '\0';
    if(link("bd", name) != 0){
      printf("%s: bigdir i=%d link(bd, %s) failed\n", s, i, name);
      exit(1);
    }
  }

  unlink("bd");
  for(i = 0; i < N; i++){
    name[0] = 'x';
    name[1] = '0' + (i / 64);
    name[2] = '0' + (i % 64);
    name[3] = '\0';
    if(unlink(name) != 0){
      printf("%s: bigdir unlink failed", s);
      exit(1);
    }
  }
}

```

## `manywrites()`
> concurrent writes to try to provoke deadlock in the virtio disk driver.

```c
void
manywrites(char *s)
{
  int nchildren = 4;
  int howmany = 30; // increase to look for deadlock
  
  for(int ci = 0; ci < nchildren; ci++){
    int pid = fork();
    if(pid < 0){
      printf("fork failed\n");
      exit(1);
    }

    if(pid == 0){
      char name[3];
      name[0] = 'b';
      name[1] = 'a' + ci;
      name[2] = '\0';
      unlink(name);
      
      for(int iters = 0; iters < howmany; iters++){
        for(int i = 0; i < ci+1; i++){
          int fd = open(name, O_CREATE | O_RDWR);
          if(fd < 0){
            printf("%s: cannot create %s\n", s, name);
            exit(1);
          }
          int sz = sizeof(buf);
          int cc = write(fd, buf, sz);
          if(cc != sz){
            printf("%s: write(%d) ret %d\n", s, sz, cc);
            exit(1);
          }
          close(fd);
        }
        unlink(name);
      }

      unlink(name);
      exit(0);
    }
  }

  for(int ci = 0; ci < nchildren; ci++){
    int st = 0;
    wait(&st);
    if(st != 0)
      exit(st);
  }
  exit(0);
}

```

## `badwrite()`
> regression test. does write() with an invalid buffer pointer cause a block to be allocated for a file that is then not freed when the file is deleted? if the kernel has this bug, it will panic: balloc: out of blocks. assumed_free may need to be raised to be more than the number of free blocks. this test takes a long time.

```c
void
badwrite(char *s)
{
  int assumed_free = 600;
  
  unlink("junk");
  for(int i = 0; i < assumed_free; i++){
    int fd = open("junk", O_CREATE|O_WRONLY);
    if(fd < 0){
      printf("open junk failed\n");
      exit(1);
    }
    write(fd, (char*)0xffffffffffL, 1);
    close(fd);
    unlink("junk");
  }

  int fd = open("junk", O_CREATE|O_WRONLY);
  if(fd < 0){
    printf("open junk failed\n");
    exit(1);
  }
  if(write(fd, "x", 1) != 1){
    printf("write failed\n");
    exit(1);
  }
  close(fd);
  unlink("junk");

  exit(0);
}

```

## `execout()`
> test the exec() code that cleans up if it runs out of memory. it's really a test that such a condition doesn't cause a panic.

```c
void
execout(char *s)
{
  for(int avail = 0; avail < 15; avail++){
    int pid = fork();
    if(pid < 0){
      printf("fork failed\n");
      exit(1);
    } else if(pid == 0){
      // allocate all of memory.
      while(1){
        char *a = sbrk(PGSIZE);
        if(a == SBRK_ERROR)
          break;
        *(a + PGSIZE - 1) = 1;
      }

      // free a few pages, in order to let exec() make some
      // progress.
      for(int i = 0; i < avail; i++)
        sbrk(-PGSIZE);
      
      close(1);
      char *args[] = { "echo", "x", 0 };
      exec("echo", args);
      exit(0);
    } else {
      wait((int*)0);
    }
  }

  exit(0);
}

```

## `diskfull()`
> can the kernel tolerate running out of disk space?

```c
void
diskfull(char *s)
{
  int fi;
  int done = 0;

  unlink("diskfulldir");
  
  for(fi = 0; done == 0 && '0' + fi < 0177; fi++){
    char name[32];
    name[0] = 'b';
    name[1] = 'i';
    name[2] = 'g';
    name[3] = '0' + fi;
    name[4] = '\0';
    unlink(name);
    int fd = open(name, O_CREATE|O_RDWR|O_TRUNC);
    if(fd < 0){
      // oops, ran out of inodes before running out of blocks.
      printf("%s: could not create file %s\n", s, name);
      done = 1;
      break;
    }
    for(int i = 0; i < MAXFILE; i++){
      char buf[BSIZE];
      if(write(fd, buf, BSIZE) != BSIZE){
        done = 1;
        close(fd);
        break;
      }
    }
    close(fd);
  }

  // now that there are no free blocks, test that dirlink()
  // merely fails (doesn't panic) if it can't extend
  // directory content. one of these file creations
  // is expected to fail.
  int nzz = 128;
  for(int i = 0; i < nzz; i++){
    char name[32];
    name[0] = 'z';
    name[1] = 'z';
    name[2] = '0' + (i / 32);
    name[3] = '0' + (i % 32);
    name[4] = '\0';
    unlink(name);
    int fd = open(name, O_CREATE|O_RDWR|O_TRUNC);
    if(fd < 0)
      break;
    close(fd);
  }

  // this mkdir() is expected to fail.
  if(mkdir("diskfulldir") == 0)
    printf("%s: mkdir(diskfulldir) unexpectedly succeeded!\n", s);

  unlink("diskfulldir");

  for(int i = 0; i < nzz; i++){
    char name[32];
    name[0] = 'z';
    name[1] = 'z';
    name[2] = '0' + (i / 32);
    name[3] = '0' + (i % 32);
    name[4] = '\0';
    unlink(name);
  }

  for(int i = 0; '0' + i < 0177; i++){
    char name[32];
    name[0] = 'b';
    name[1] = 'i';
    name[2] = 'g';
    name[3] = '0' + i;
    name[4] = '\0';
    unlink(name);
  }
}

```

## `outofinodes()`
> [Geen commentaar in broncode]: Test het gedrag van het bestandssysteem wanneer de limiet van het aantal inodes is bereikt door een groot aantal kleine bestanden aan te maken.

```c

void
outofinodes(char *s)
{
  int nzz = 32*32;
  for(int i = 0; i < nzz; i++){
    char name[32];
    name[0] = 'z';
    name[1] = 'z';
    name[2] = '0' + (i / 32);
    name[3] = '0' + (i % 32);
    name[4] = '\0';
    unlink(name);
    int fd = open(name, O_CREATE|O_RDWR|O_TRUNC);
    if(fd < 0){
      // failure is eventually expected.
      break;
    }
    close(fd);
  }

  for(int i = 0; i < nzz; i++){
    char name[32];
    name[0] = 'z';
    name[1] = 'z';
    name[2] = '0' + (i / 32);
    name[3] = '0' + (i % 32);
    name[4] = '\0';
    unlink(name);
  }
}

```

## `countfree()`
> use sbrk() to count how many free physical memory pages there are.

```c
int
countfree()
{
  int n = 0;
  uint64 sz0 = (uint64)sbrk(0);
  while(1){
    char *a = sbrk(PGSIZE);
    if(a == SBRK_ERROR){
      break;
    }
    n += 1;
  }
  sbrk(-((uint64)sbrk(0) - sz0));  
  return n;
}

```

## `main()`
> [Geen commentaar in broncode]: De entry-point voor de user tests; verwerkt command-line argumenten om specifieke tests uit te voeren of alle tests in een lus te draaien.

```c

int
main(int argc, char *argv[])
{
  int continuous = 0;
  int quick = 0;
  char *justone = 0;

  if(argc == 2 && strcmp(argv[1], "-q") == 0){
    quick = 1;
  } else if(argc == 2 && strcmp(argv[1], "-c") == 0){
    continuous = 1;
  } else if(argc == 2 && strcmp(argv[1], "-C") == 0){
    continuous = 2;
  } else if(argc == 2 && argv[1][0] != '-'){
    justone = argv[1];
  } else if(argc > 1){
    printf("Usage: usertests [-c] [-C] [-q] [testname]\n");
    exit(1);
  }
  if (drivetests(quick, continuous, justone)) {
    exit(1);
  }
  printf("ALL TESTS PASSED\n");
  exit(0);
}

```
