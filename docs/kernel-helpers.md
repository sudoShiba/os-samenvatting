# Kernel Helpers

The xv6 kernel provides several utility functions to simplify common tasks.

## Argument Retrieval
These functions are used in system call implementations to safely retrieve arguments passed from user space.
- **`argint(int n, int *ip)`**: Gets the $n$-th integer argument.
- **`argaddr(int n, uint64 *ip)`**: Gets the $n$-th pointer argument (as a `uint64`).
- **`argstr(int n, char *buf, int max)`**: Gets the $n$-th string argument.

## Memory Access
- **`copyin(pagetable_t pagetable, char *dst, uint64 srcva, uint64 len)`**: Copies data from user space to kernel space.
- **`copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)`**: Copies data from kernel space to user space.
- **`walkaddr(pagetable_t pagetable, uint64 va)`**: Translates a user virtual address to a physical address.

## Memory Management
Common functions for handling pages and page tables:
- **`kalloc(void)`**: Geeft een vrije fysieke page terug (4096 bytes).
- **`memset(void*, int, uint)`**: Zet een blok geheugen op een bepaalde waarde (handig voor het wissen van nieuwe pages).
- **`mappages(pagetable_t, uint64 va, uint64 size, uint64 pa, int perm)`**: Voegt PTE's toe aan een pagetable om een virtueel adresbereik naar een fysiek bereik te mappen.
- **`uvmcopy(pagetable_t old, pagetable_t new, uint64 size)`**: Kopieert zowel de pagetable als het fysieke geheugen van een parent naar een kind (gebruikt in `fork`).
- **`uvmunmap(pagetable_t, uint64 va, uint64 npages, int do_free)`**: Verwijdert $n$ pages van een mapping. Indien `do_free` 1 is, wordt ook het fysieke geheugen vrijgegeven.
- **`uvmfree(pagetable_t, uint64 size)`**: Bevrijdt de pagetable van een proces en het bijbehorende fysieke geheugen.
- **`uvmcreate(void)`**: Creëert een lege user page table.

### Page Table Walking
- **`walk(pagetable_t, uint64 va, int alloc)`**: Simuleert de walk die de MMU doet. Returnt een pointer naar de leaf PTE. Indien `alloc` 1 is, kunnen ontbrekende pagetables in de boom worden gealloceerd.
- **`walkaddr(pagetable_t, uint64 va)`**: Returnt het fysieke adres van de PTE voor een virtueel adres. Checkt ook of de page gemapt is in user space.

## Output
- **`printf(char *fmt, ...)`**: Prints formatted text to the console (kernel only).
- **`panic(char *s)`**: Prints an error message and halts the kernel. Use only for fatal errors.

## Locking
- **`acquire(struct spinlock *lk)`**: Acquires a spinlock.
- **`release(struct spinlock *lk)`**: Releases a spinlock.
- **`sleep(void *chan, struct spinlock *lk)`**: Relinquishes the CPU and sleeps until `wakeup(chan)` is called.
- **`wakeup(void *chan)`**: Wakes up all processes sleeping on `chan`.
