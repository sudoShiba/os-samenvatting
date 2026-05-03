# `kalloc.c`

Dit bestand bevat de implementaties voor `kernel/kalloc.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `kinit()`
> [Geen commentaar in broncode]: Initialiseert de fysieke geheugenbeheerder door de kmem-lock aan te maken en het vrije geheugenbereik toe te voegen aan de freelist.

```c

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
}

```

## `freerange()`
> [Geen commentaar in broncode]: Voegt een reeks fysieke pagina's toe aan de lijst van vrije pagina's.

```c

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}

```

## `kfree()`
> Free the page of physical memory pointed at by pa, which normally should have been returned by a call to kalloc().  (The exception is when initializing the allocator; see kinit above.)

```c
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}

```

## `kalloc()`
> Allocate one 4096-byte page of physical memory. Returns a pointer that the kernel can use. Returns 0 if the memory cannot be allocated.

```c
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

```

