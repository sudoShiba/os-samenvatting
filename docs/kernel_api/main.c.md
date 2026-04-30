# `main.c`

Dit bestand bevat de implementaties voor `kernel/main.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `main()`
> start() jumps here in supervisor mode on all CPUs.

```c
void
main()
{
  if(cpuid() == 0){
    consoleinit();
    printfinit();
    printf("\n");
    printf("xv6 kernel is booting\n");
    printf("\n");
    kinit();         // physical page allocator
    kvminit();       // create kernel page table
    kvminithart();   // turn on paging
    procinit();      // process table
    trapinit();      // trap vectors
    trapinithart();  // install kernel trap vector
    plicinit();      // set up interrupt controller
    plicinithart();  // ask PLIC for device interrupts
    binit();         // buffer cache
    iinit();         // inode table
    fileinit();      // file table
    virtio_disk_init(); // emulated hard disk
    userinit();      // first user process
    __sync_synchronize();
    started = 1;
  } else {
    while(started == 0)
      ;
    __sync_synchronize();
    printf("hart %d starting\n", cpuid());
    kvminithart();    // turn on paging
    trapinithart();   // install kernel trap vector
    plicinithart();   // ask PLIC for device interrupts
  }

  scheduler();        
}

```

## `halt()`
> Geen specifieke commentaar in de broncode.

```c

void halt(void)
{
  *(uint32*)SIFIVE_TEST = 0x5555;
}

```

## `sys_halt()`
> Geen specifieke commentaar in de broncode.

```c

uint64 sys_halt(void)
{
  halt();
  return 0;
}

```

