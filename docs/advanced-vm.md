# Lab 7: Advanced Virtual Memory (Exam Target)

This lab focuses on complex memory optimizations: Deduplication, COW Null Page, and File-backed mmap.

## 1. Memory Deduplication (`sys_dedup`)
Merging identical physical pages into a single COW page.

**Kernel Implementation (`kernel/sysdedup.c`):**
```c
void sys_dedup(void) {
  struct proc *p = myproc();
  // 1. Scan heap pages
  parse_heap_pages(p->pagetable, p->sbrk_base, p->sz/PGSIZE, heap_pages, MAX_HEAP_PAGES);

  // 2. Compare pages
  for (int i = 0; i < MAX_HEAP_PAGES; i++) {
    for (int j = i+1; j < MAX_HEAP_PAGES; j++) {
      if (memcmp((void*) heap_pages[i].pa, (void*) heap_pages[j].pa, PGSIZE) == 0) {
        // 3. Remap duplicated page (va_dst) to master page (va_src)
        uvmdedup(p->pagetable, heap_pages[i].va, heap_pages[j].va);
      }
    }
  }
}

void uvmdedup(pagetable_t pt, uint64 va_src, uint64 va_dst) {
  pte_t *pte_src = walk(pt, va_src, 0);
  pte_t *pte_dst = walk(pt, va_dst, 0);
  uint64 pa_src = PTE2PA(*pte_src);
  
  // Set COW, Clear Write for both
  *pte_src = (*pte_src & ~PTE_W) | PTE_COW;
  *pte_dst = PA2PTE(pa_src) | (PTE_FLAGS(*pte_dst) & ~PTE_W) | PTE_COW;
  
  kref((void*)pa_src); // Increase refcount
  // Old pa_dst will be freed when process dies or write occurs
}
```

## 2. File-backed Memory Mapping
Extending `mmap` to map files. This requires lazy loading.

**VMA Structure in `struct proc`:**
```c
struct vma {
  int valid;
  uint64 addr;
  int length;
  struct file *f;
  int off;
  int prot;
};
```

**Fault Handler logic:**
```c
uint64 handle_mmap_file_fault(uint64 va) {
  struct vma *v = find_vma(va);
  char *mem = kalloc();
  
  // Read from file at the correct offset
  ilock(v->f->ip);
  readi(v->f->ip, 0, (uint64)mem, v->off + (PGROUNDDOWN(va) - v->addr), PGSIZE);
  iunlock(v->f->ip);
  
  mappages(myproc()->pagetable, PGROUNDDOWN(va), PGSIZE, (uint64)mem, v->prot | PTE_U);
  return (uint64)mem;
}
```

## 3. COW Null Page
Map `0x0` to a dedicated `zero_page` in `proc_pagetable()`:
```c
extern char zero_page[]; // A page of zeros defined in kernel

mappages(pagetable, 0, PGSIZE, (uint64)zero_page, PTE_R | PTE_U | PTE_COW);
kref(zero_page);
```
When a write happens to `0x0`, the `vmfault` COW logic (from Lab 5) will automatically allocate a private page and copy the zeros, allowing the process to continue.
