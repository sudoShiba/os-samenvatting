# vDSO Toevoegen (Walkthrough)

vDSO (Virtual Dynamically-linked Shared Object) wordt gebruikt om data uit de kernel direct beschikbaar te maken in user space zonder de overhead van een system call.

## Stap 1: Variabele in een eigen sectie plaatsen
Plaats de variabele die je wilt delen in een aparte geheugensectie. Dit zorgt ervoor dat we deze later specifiek kunnen mappen. Doe dit bijvoorbeeld in `kernel/trap.c`:

```c
uint ticks __attribute__((section(".vdso")));
```

## Stap 2: Linker aanpassen (`kernel/kernel.ld`)
We moeten de linker vertellen waar deze sectie moet komen in het fysieke geheugen. Zorg dat de sectie exact één page (4096 bytes) groot is en net boven het einde van de kernel (`end`) wordt geplaatst.

```ld
  .vdso : {
    . = ALIGN(0x1000); /* Uitlijnen op page boundary */
    _vdso_start = .;   /* Maak pointer naar startadres */
    *(.vdso);          /* Plaats variabelen uit de .vdso sectie hier */
    . = ALIGN(0x1000); /* Lijn einde uit */
    ASSERT(. - _vdso_start == 0x1000, "vdso groter dan 1 page");
  }
```

## Stap 3: Mappen in elk user proces (`kernel/proc.c`)
Haal de startpointer op via `extern char _vdso_start[]` en voeg de mapping toe aan de functie `proc_pagetable()`. De virtuele locatie is meestal net onder de `TRAPFRAME`.

```c
extern char _vdso_start[];

// In proc_pagetable():
uint64 offset = TRAPFRAME - PGSIZE;
if(mappages(pagetable, offset, PGSIZE, (uint64)_vdso_start, PTE_R | PTE_U) < 0){
  // Mislukt? Unmap dan de eerder gemapte pages en free de pagetable!
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  return 0;
}
```
**Cruciaal:** Gebruik enkel de permissies `PTE_R | PTE_U`. Geef **nooit** `PTE_W` (Write) permissie mee, anders kan een malafide user programma de systeemtijd aanpassen!

## Stap 4: Mapping verwijderen (`kernel/proc.c`)
Wanneer een proces stopt, moet de mapping weer netjes verwijderd worden in `proc_freepagetable()`.

```c
// Laatste argument is 0: unmap de PTE, maar free de fysieke kernel page NIET!
uvmunmap(pagetable, TRAPFRAME - PGSIZE, 1, 0);
```

## Stap 5: Gebruik in User Space
Definieer het adres in een header file (bv. `user/user.h`) en lees de waarde direct uit het geheugen:

```c
#define VDSOPAGE (TRAPFRAME - PGSIZE)

uint uptime() {
  return *(uint*)VDSOPAGE;
}
```
