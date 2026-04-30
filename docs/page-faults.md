# Lab 5: Copy-on-Write (Expert Depth)

Copy-on-Write (COW) is a fundamental optimization. In the exam, you'll need to handle reference counts and trap-based page allocation.

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
