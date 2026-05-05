# RISC-V Supervisor Exception Codes (`scause`)

Wanneer de kernel een trap afhandelt, bevat het `scause` register een code die aangeeft waarom de trap is opgetreden. Deze tabel is essentieel voor het debuggen van traps en het implementeren van functies in `kernel/trap.c`.

Het meest significante bit (MSB) van `scause` geeft aan of de trap een **Interrupt** (bit = 1) of een **Exception** (bit = 0) was.

---

## 🛑 Exceptions (Interrupt bit = 0)
Exceptions treden op als gevolg van een specifieke instructie of een foutieve operatie.

| Code | Naam | Beschrijving | Relevant voor Lab... |
| :--- | :--- | :--- | :--- |
| **0** | Instruction address misaligned | Program counter (`sepc`) wijst naar ongeldig adres. | - |
| **1** | Instruction access fault | Geen leesrechten op de instructiepagina. | - |
| **2** | Illegal instruction | CPU herkent de instructie niet (bijv. kernel instructie in user mode). | - |
| **3** | Breakpoint | `ebreak` instructie aangeroepen. | - |
| **4** | Load address misaligned | Data lezen van een niet-gealigned adres. | - |
| **5** | Load access fault | Geen leesrechten op de geheugenlocatie. | - |
| **6** | Store/AMO address misaligned | Data schrijven naar een niet-gealigned adres. | - |
| **7** | Store/AMO access fault | Geen schrijfrechten op de geheugenlocatie. | - |
| **8** | Environment call from U-mode | **System Call!** Het programma vraagt kernel assistentie. | **Lab 2** |
| **9** | Environment call from S-mode | Kernel vraagt assistentie van een hoger niveau. | - |
| **12** | Instruction page fault | Pagina niet gemapt of geen rechten (bijv. executie van data). | **Lab 5** |
| **13** | Load page fault | Pagina niet gemapt (bijv. **Lazy Allocation** of **COW**). | **Lab 5/7** |
| **15** | Store/AMO page fault | Pagina niet gemapt of schrijven naar read-only (bijv. **COW**). | **Lab 5/7** |

---

## ⚡ Interrupts (Interrupt bit = 1)
Interrupts treden asynchroon op (bijv. door hardware timers of externe apparaten).

| Code | Naam | Beschrijving | Relevant voor Lab... |
| :--- | :--- | :--- | :--- |
| **1** | Supervisor software interrupt | Wordt gebruikt voor communicatie tussen CPU cores. | - |
| **5** | Supervisor timer interrupt | **Timer tick!** Wordt gebruikt voor scheduling/pre-emption. | **Lab 3** |
| **9** | Supervisor external interrupt | PLIC (hardware device) vraagt aandacht (bijv. UART/Disk). | **Lab 6** |

---

## 🔍 Gebruik in de Code
In `kernel/trap.c` kun je de code als volgt uitlezen:

```c
void
usertrap(void)
{
  uint64 scause = r_scause();

  if(scause == 8){
    // System call
    syscall();
  } else if(scause == 13 || scause == 15){
    // Page fault!
    handle_page_fault(r_stval());
  } else if((scause & 0x8000000000000000L) && (scause & 0xff) == 5){
    // Timer interrupt
    yield();
  } else {
    // Onbekende fout -> panic of kill process
  }
}
```

> **Tip:** Gebruik de `0x8000000000000000L` bitmask om te controleren of de interrupt bit (MSB) op 1 staat.
