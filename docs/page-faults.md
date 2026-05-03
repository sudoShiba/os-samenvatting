# Lab 5: Page Faults & Copy-on-Write (Expert Depth)

Een **Page Fault** is een hardware trap die optreedt als een programma geheugen opvraagt dat de MMU niet kan vertalen of waarvoor de permissies niet kloppen.

## 0. Page Fault Types & Scenario's
Bij een trap vult de CPU twee registers in:
- **`scause` (Supervisor Cause):** Vertelt waarom de trap gebeurde.
    - **Code 12:** Instruction Page Fault (CPU kan instructie op dit adres niet laden).
    - **Code 13:** Load Page Fault (CPU kan data op dit adres niet lezen).
    - **Code 15:** Store Page Fault (CPU kan data op dit adres niet schrijven).
- **`stval` (Supervisor Trap Value):** Bevat het virtuele adres dat de fout veroorzaakte.

### Veelvoorkomende Scenario's
1.  **Segmentation Fault:** Het programma probeert geheugen te lezen dat niet gemapt is (`PTE_V=0`) of probeert te schrijven naar read-only geheugen (`PTE_W=0`).
2.  **Demand Paging / Lazy Allocation:** Het programma vraagt om RAM via `sbrk`, maar het OS stelt de allocatie uit.
3.  **Copy-on-Write (CoW):** Fork deelt pagina's read-only en kopieert ze pas bij een Store Page Fault.

## 1. Reference Counting (`kernel/kalloc.c`)
De allocator moet bijhouden hoeveel pagetables naar een fysieke pagina verwijzen met `kincref`.
```c
int refcount[PHYSTOP / PGSIZE];

void kincref(void *pa)
{
  acquire(&kmem.lock);
  refcount[PGROUNDDOWN((uint64)pa - KERNBASE) / PGSIZE]++;
  release(&kmem.lock);
}

int kdecref(void *pa)
{
  acquire(&kmem.lock);
  int count = --refcount[PGROUNDDOWN((uint64)pa - KERNBASE) / PGSIZE];
  release(&kmem.lock);
  return count;
}

void
kfree(void *pa)
{
  struct run *r;
  if (kdecref(pa) > 0)
    return;
  // ... rest van standaard kfree ...
}

void *
kalloc(void)
{
  struct run *r;
  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r) {
    kmem.freelist = r->next;
    refcount[((uint64)r - KERNBASE) / PGSIZE] = 1;
  }
  release(&kmem.lock);
  // ...
}
```

## 2. Modifying `uvmcopy` (COW Fork in `kernel/vm.c`)
In plaats van te kopiëren, delen we de pagina en markeren deze als `PTE_COW`.
```c
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0 || (*pte & PTE_V) == 0)
      continue;

    pa = PTE2PA(*pte);
    if (*pte & PTE_U) {
      if (*pte & PTE_W) {
        *pte = (*pte & ~PTE_W) | PTE_COW;
      }   
      if(mappages(new, i, PGSIZE, pa, PTE_FLAGS(*pte)) != 0)
        goto err;
      kincref((void *)pa);
    } else {
      // Kopieer kernel/stack pagina's zoals normaal
      char *mem = kalloc();
      memmove(mem, (char*)pa, PGSIZE);
      if(mappages(new, i, PGSIZE, (uint64)mem, PTE_FLAGS(*pte)) != 0) goto err;
    }
  }
  return 0;
 err:
  uvmunmap(new, 0, i / PGSIZE, 1);
  return -1;
}
```

## 3. Handling the Fault (`vmfault` in `kernel/vm.c`)
Deze functie handelt zowel Lazy Allocation als COW af.
```c
uint64
vmfault(pagetable_t pagetable, uint64 va, int read)
{
  struct proc *p = myproc();
  if (va >= p->sz) return 0;
  va = PGROUNDDOWN(va);
  pte_t *pte = walk(pagetable, va, 0);

  uint64 mem = 0;
  int new_flags = 0;

  if (pte && *pte & PTE_V){
    if (!(*pte & PTE_W) && *pte & PTE_COW) {
      // COW case
      mem = (uint64)kalloc();
      if(mem == 0) return 0;
      memmove((void *)mem, (void *)PTE2PA(*pte), PGSIZE);
      new_flags = (PTE_FLAGS(*pte) & ~PTE_COW) | PTE_W;
      uvmunmap(pagetable, va, 1, 1); // decref oude pagina
    } else {
      return 0; // Ongeldige fault op aanwezige pagina
    }
  } else {
    // Lazy allocation case
    mem = (uint64) kalloc();
    if(mem == 0) return 0;
    memset((void *) mem, 0, PGSIZE);
    new_flags = PTE_W|PTE_U|PTE_R;
  }

  if (mappages(p->pagetable, va, PGSIZE, mem, new_flags) != 0) {
    kfree((void *)mem);
    return 0;
  }
  return mem;
}
```

## 4. Updates to `copyout` (`kernel/vm.c`)
Cruciaal: de COW-check moet in een `else` staan na de lazy-check.
```c
int
copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)
{
  // ... loop ...
  pa0 = walkaddr(pagetable, va0);
  if(pa0 == 0) {
    if((pa0 = vmfault(pagetable, va0, 0)) == 0) return -1;
  } else {
    // Check voor COW op een reeds gemapte pagina
    pte = walk(pagetable, va0, 0);
    if(*pte & PTE_COW) {
      if((pa0 = vmfault(pagetable, va0, 0)) == 0) return -1;
    }
  }
  // ... memmove ...
}
```
