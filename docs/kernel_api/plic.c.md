# `plic.c`

Dit bestand bevat de implementaties voor `kernel/plic.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `plicinit()`
>  the riscv Platform Level Interrupt Controller (PLIC). 

```c

void
plicinit(void)
{
  // set desired IRQ priorities non-zero (otherwise disabled).
  *(uint32*)(PLIC + UART0_IRQ*4) = 1;
  *(uint32*)(PLIC + VIRTIO0_IRQ*4) = 1;
}

```

## `plicinithart()`
> Geen specifieke commentaar in de broncode.

```c

void
plicinithart(void)
{
  int hart = cpuid();
  
  // set enable bits for this hart's S-mode
  // for the uart and virtio disk.
  *(uint32*)PLIC_SENABLE(hart) = (1 << UART0_IRQ) | (1 << VIRTIO0_IRQ);

  // set this hart's S-mode priority threshold to 0.
  *(uint32*)PLIC_SPRIORITY(hart) = 0;
}

```

## `plic_claim()`
> ask the PLIC what interrupt we should serve.

```c
int
plic_claim(void)
{
  int hart = cpuid();
  int irq = *(uint32*)PLIC_SCLAIM(hart);
  return irq;
}

```

## `plic_complete()`
> tell the PLIC we've served this IRQ.

```c
void
plic_complete(int irq)
{
  int hart = cpuid();
  *(uint32*)PLIC_SCLAIM(hart) = irq;
}

```

