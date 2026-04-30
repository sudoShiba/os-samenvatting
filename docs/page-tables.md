# Lab 4: Page Tables (Expert Depth)

Page tables are a core component of the exam. You must understand how to walk them, map pages, and implement vDSO.

## 1. `vmprintmappings` (Walking the Tree)
Recursive function to print all leaf nodes in a page table.
**Implementation in `kernel/vm.c`:**
```c
static void print_pagetable(pagetable_t pagetable, uint64 base_va, int level)
{
  for (uint64 pte_i = 0; pte_i < 512; ++pte_i) {
    pte_t* pte = &pagetable[pte_i];
    uint64 pte_va = base_va | (pte_i << PXSHIFT(level));
    uint64 pte_pa = PTE2PA(*pte);
    uint pte_flags = PTE_FLAGS(*pte);

    if (pte_flags & PTE_V) {
      if (level == 0) {
        char mode = (pte_flags & PTE_U) ? 'U' : 'S';
        char r    = (pte_flags & PTE_R) ? 'r' : '-';
        char w    = (pte_flags & PTE_W) ? 'w' : '-';
        char x    = (pte_flags & PTE_X) ? 'x' : '-';
        printf("%p -> %p, mode=%c, perms=%c%c%c\n", (void *)pte_va, (void *)pte_pa, mode, r, w, x);
      } else {
        pagetable_t next_pagetable = (pagetable_t)pte_pa;
        print_pagetable(next_pagetable, pte_va, level - 1);
      }
    }
  }
}
```

## 2. vDSO Implementation
Mapping a shared read-only page to speed up system calls like `uptime`.

### Linker Script (`kernel/kernel.ld`)
The `.vdso` section must be page-aligned and exactly one page.
```ld
  .vdso : {            /* Start an "output section" named .vdso */
    . = ALIGN(0x1000); /* Align (.) on a page boundary */
    _vdso_start = .;   /* Define _vdso_start */
    *(.vdso);          /* Put all input .vdso sections here */
    . = ALIGN(0x1000); /* Align on page boundary */
    ASSERT(. - _vdso_start == 0x1000, "error: vdso section exceeds one page");
  }
```

### Global Variables (**`kernel/trap.c`**)
```c
uint ticks __attribute__((section(".vdso")));
```

### Mapping in **`kernel/proc.c`**
In `proc_pagetable()`:
```c
extern char _vdso_start[];

// map the vDSO page just below the trapframe page
if(mappages(pagetable, VDSOPAGE, PGSIZE, (uint64)_vdso_start, PTE_R | PTE_U) < 0){
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmfree(pagetable, 0);
  return 0;
}
```

### User Library (**`user/ulib.c`** or **`user/fastuptime.c`**)
```c
uint fastuptime(void) {
  return *((uint *) VDSOPAGE);
}
```

## 3. Basic `mmap`
Implement anonymous private memory mapping.
**`sys_mmap` logic:**
```c
uint64 sys_mmap(void) {
  uint64 vaddr;
  int perms;
  argaddr(0, &vaddr);
  argint(1, &perms);

  struct proc *p = myproc();
  if(vaddr >= MAXVA || vaddr % PGSIZE != 0) return -1;
  
  // Check if already mapped
  if(walkaddr(p->pagetable, vaddr) != 0) return -1;

  char *mem = kalloc();
  if(mem == 0) return -1;
  memset(mem, 0, PGSIZE);

  if(mappages(p->pagetable, vaddr, PGSIZE, (uint64)mem, perms | PTE_U) != 0){
    kfree(mem); return -1;
  }
  // Store mmapped address in p->mmapped array for cleanup
  p->mmapped[p->mmapcount++] = vaddr;
  return 0;
}
```
**Important:** Don't forget `uvmunmap` in `freeproc` and copying mappings in `fork`.
