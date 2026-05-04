# vDSO Toevoegen (Walkthrough)

vDSO (Virtual Dynamically-linked Shared Object) wordt gebruikt om data uit de kernel direct beschikbaar te maken in user space zonder de overhead van een system call.

## Stap 1: Variabele in een eigen sectie plaatsen
Plaats de variabele die je wilt delen in een aparte geheugensectie in `kernel/trap.c`:
```c
uint ticks __attribute__((section(".vdso")));
```

## Stap 2: Linker aanpassen (`kernel/kernel.ld`)
```ld
  .vdso : {
    . = ALIGN(0x1000);
    _vdso_start = .;
    *(.vdso);
    . = ALIGN(0x1000);
    ASSERT(. - _vdso_start == 0x1000, "vdso groter dan 1 page");
  }
```

## Stap 3: Adres definiëren in `kernel/memlayout.h`
Conform de architectuur van xv6 definiëren we het virtuele adres in de kernel layout:
```c
#define VDSOPAGE (TRAPFRAME - PGSIZE)
```

## Stap 4: Mappen in elk user proces (`kernel/proc.c`)
In `proc_pagetable()`:
```c
extern char _vdso_start[];

if(mappages(pagetable, VDSOPAGE, PGSIZE, (uint64)_vdso_start, PTE_R | PTE_U) < 0){
  // ... error handling ...
}
```

## Stap 5: Mapping verwijderen (`kernel/proc.c`)
In `proc_freepagetable()`:
```c
uvmunmap(pagetable, VDSOPAGE, 1, 0);
```

## Stap 6: Gebruik in User Space
In `user/user.h` (of een specifieke user file):
```c
uint uptime() {
  return *(uint*)VDSOPAGE;
}
```
