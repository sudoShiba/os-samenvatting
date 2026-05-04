# Boilerplate & Templates

Dit document bevat veelgebruikte code-skeletten voor xv6-ontwikkeling. Kopieer en pas deze aan voor je eigen implementaties.

---

## 1. User-Space Programma Template
Gebruik dit als basis voor elk nieuw user-programma in `user/`.

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  // 1. Argument Check
  if(argc < 2){
    fprintf(2, "Usage: %s arg1 [arg2...]\n", argv[0]);
    exit(1);
  }

  // 2. Variabelen ophalen
  char *path = argv[1];
  // int val = atoi(argv[2]);

  // 3. Logica (bijv. syscall aanroepen)
  if(uptime() < 0){
    fprintf(2, "%s: logic failed\n", argv[0]);
    exit(1);
  }

  // 4. Altijd eindigen met exit()
  exit(0);
}
```

---

## 2. Kernel-Side System Call (File System)
Voor syscalls die bestanden aanpassen, meestal in `kernel/sysfile.c`.

```c
uint64
sys_my_fs_call(void)
{
  char path[MAXPATH];
  int val;
  struct inode *ip;

  // 1. Argumenten ophalen
  if(argstr(0, path, MAXPATH) < 0 || argint(1, &val) < 0)
    return -1;

  // 2. Transactie starten
  begin_op();

  // 3. Inode opzoeken
  if((ip = namei(path)) == 0){
    end_op();
    return -1;
  }

  ilock(ip);
  
  // 4. Logica & Persistence
  // ip->something = val;
  iupdate(ip);

  iunlockput(ip);
  end_op();

  return 0;
}
```

---

## 3. Kernel-Side System Call (Proces/Geheugen)
Voor syscalls die processen of CPU-staat aanpassen, meestal in `kernel/sysproc.c`.

```c
uint64
sys_my_proc_call(void)
{
  int n;
  struct proc *p = myproc();

  // 1. Argumenten ophalen
  if(argint(0, &n) < 0)
    return -1;

  // 2. Exclusieve toegang tot proces-data (indien nodig)
  acquire(&p->lock);
  
  // 3. Logica
  // p->sz += n;
  
  release(&p->lock);

  return 0;
}
```

---

## 4. User-Kernel Data Transfer
Als je pointers (buffers) doorgeeft aan een syscall, moet je `copyin` of `copyout` gebruiken.

**Template voor `copyin` (User -> Kernel):**
```c
uint64 buf_addr;
char kbuf[128];
argaddr(0, &buf_addr);

if(copyin(myproc()->pagetable, kbuf, buf_addr, sizeof(kbuf)) < 0)
  return -1;
```

**Template voor `copyout` (Kernel -> User):**
```c
uint64 buf_addr;
char *msg = "Hello";
argaddr(1, &buf_addr);

if(copyout(myproc()->pagetable, buf_addr, msg, strlen(msg)+1) < 0)
  return -1;
```

---

## 5. Geheugen Allocatie (Kernel)
Basispatroon voor het aanmaken van een nieuwe pagina (bijv. voor een page table of data).

```c
char *mem;
if((mem = kalloc()) == 0)
  return -1; // Out of memory

memset(mem, 0, PGSIZE); // Altijd op 0 zetten voor veiligheid
```

---

## 6. Directory Iteratie (User Space)
Boilerplate om door bestanden in een map te lussen (gebaseerd op `ls.c`).

```c
struct dirent de;
struct stat st;
int fd = open(".", 0);

while(read(fd, &de, sizeof(de)) == sizeof(de)){
  if(de.inum == 0) continue;
  // de.name bevat de bestandsnaam
}
close(fd);
```
