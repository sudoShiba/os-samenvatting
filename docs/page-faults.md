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
1.  **Segmentation Fault:** Het programma probeert geheugen te lezen dat niet gemapt is (`PTE_V=0`) of probeert te schrijven naar read-only geheugen (`PTE_W=0`) zonder dat dit een geoptimaliseerd scenario is (zoals CoW). De kernel zal het proces in dit geval killen.
2.  **Demand Paging / Lazy Allocation:** Het programma vraagt om RAM via `sbrk`, maar het OS past enkel `p->sz` aan zonder echt pages te mappen. Pas wanneer het proces de page echt aanraakt, krijgt het een fault. De kernel ziet dat het adres binnen `p->sz` ligt, alloceert dan pas een page met `kalloc()`, mapt deze met `mappages()` en laat de CPU de instructie opnieuw proberen.
3.  **Swapping:** De page bestaat, maar werd tijdelijk naar de harde schijf geschreven omdat het RAM vol was. Het OS pauzeert het proces, laadt de data van disk naar RAM, updatet de PTE en hervat het proces. (Standaard xv6 doet dit niet).
4.  **Copy-on-Write (CoW):** Zie sectie hieronder.

## 1. Reference Counting (`kernel/kalloc.c`)
Modify the allocator to track how many page tables point to a physical page.
```c
int refcount[PHYSTOP / PGSIZE];

void kref(void *pa) {
  acquire(&kmem.lock);
  refcount[PGROUNDDOWN((uint64)pa - KERNBASE) / PGSIZE]++;
  release(&kmem.lock);
}

int kdecref(void *pa) {
  acquire(&kmem.lock);
  int count = --refcount[PGROUNDDOWN((uint64)pa - KERNBASE) / PGSIZE];
  release(&kmem.lock);
  return count;
}

void kfree(void *pa) {
  if (kdecref(pa) > 0) return; // Only free if last reference
  // ... standard kfree logic ...
}

void* kalloc(void) {
  // ... standard kalloc logic ...
  if(r) refcount[((uint64)r - KERNBASE) / PGSIZE] = 1;
  return (void*)r;
}
```

## 2. Modifying `uvmcopy` (COW Fork)
Instead of copying, share the page and mark it as COW.
```c
int uvmcopy(pagetable_t old, pagetable_t new, uint64 sz) {
  // ... loop over i from 0 to sz ...
  pte = walk(old, i, 0);
  pa = PTE2PA(*pte);
  
  if (*pte & PTE_U) {
    if (*pte & PTE_W) {
      *pte = (*pte & ~PTE_W) | PTE_COW; // Clear Write, Set COW
    }   
    if(mappages(new, i, PGSIZE, pa, PTE_FLAGS(*pte)) != 0) goto err;
    kref((void *)pa); // Important!
  }
}
```

## 3. Handling the Fault (`vmfault` in `kernel/vm.c`)
When a process writes to a COW page, it triggers a "Store Page Fault".
```c
uint64 vmfault(pagetable_t pagetable, uint64 va, int read) {
  struct proc *p = myproc();
  va = PGROUNDDOWN(va);
  pte_t *pte = walk(pagetable, va, 0);

  if (pte && (*pte & PTE_V) && (*pte & PTE_COW)) {
    // 1. Allocate new page
    char *mem = kalloc();
    if(mem == 0) return 0;
    
    // 2. Copy old content
    memmove(mem, (void *)PTE2PA(*pte), PGSIZE);
    
    // 3. Remap with PTE_W and without PTE_COW
    uint flags = (PTE_FLAGS(*pte) & ~PTE_COW) | PTE_W;
    uvmunmap(pagetable, va, 1, 1); // This calls kfree/kdecref
    if (mappages(p->pagetable, va, PGSIZE, (uint64)mem, flags) != 0) {
      kfree(mem); return 0;
    }
    return (uint64)mem;
  }
  return 0;
}
```

## 4. Updates to `copyout` and `copyin`
Don't forget that `copyout` (kernel writing to user) must also trigger the COW logic.
```c
int copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len) {
  // ... inside loop ...
  pte = walk(pagetable, va0, 0);
  if(*pte & PTE_COW) {
    if(vmfault(pagetable, va0, 0) == 0) return -1;
  }
  // ... standard copyout ...
}
```
**Exam Tip:** Use a custom flag for `PTE_COW`. In xv6-riscv, bit 8 or 9 (RSW bits) are usually safe to use.

## 5. Exam Example: Page Deduplication
Page deduplication is an optimization technique that identifies identical physical pages in memory and merges them into a single read-only page to save RAM.

### Implementation steps:
1.  **Identification:** A system call (e.g., `dedup()`) loops through the page tables to find physical pages in the heap.
2.  **Comparison:** The contents of these pages are compared to find duplicates.
3.  **Merging:** Duplicate pages are remapped to a single physical page.
4.  **COW Semantics:** The merged page is marked as read-only. If a process tries to write to it later, the standard Copy-on-Write logic (Step 3 above) will transparently create a new private copy for that process.

**Advantage:** Significant memory savings when multiple processes (or even the same process) have identical data in different pages.
