# Per-Process and Per-CPU State

Understanding the difference between process state and CPU state is vital for kernel development, especially for scheduling and context switching.

## Per-Process State (`struct proc`)
Defined in **`kernel/proc.h`**, the `struct proc` contains everything the kernel needs to know about a specific process.
- **`p->state`**: The current state (UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE).
- **`p->pagetable`**: The process's private page table.
- **`p->sz`**: The size of the process's memory in bytes.
- **`p->trapframe`**: A page where registers are saved when a trap occurs.
- **`p->context`**: Registers saved during a kernel-level context switch (`swtch`).
- **`p->ofile`**: An array of open files.

## Per-CPU State (`struct cpu`)
Also defined in **`kernel/proc.h`**, the `struct cpu` contains information about the physical CPU hardware.
- **`c->proc`**: A pointer to the `struct proc` currently running on this CPU.
- **`c->context`**: The kernel-level context to switch back to when the scheduler is running.

## Adding Custom State (Bookkeeping)
Soms moet je extra data onthouden over een proces (bv. hoe vaak een syscall is aangeroepen). Dit doe je als volgt:

1.  **Breid `struct proc` uit (`kernel/proc.h`):** Voeg je nieuwe variabele toe aan de struct. Gebruik `uint64` voor geheugenadressen (xv6 is een 64-bit OS).
2.  **Initialiseer in `allocproc` (`kernel/proc.c`):** Zet je nieuwe veld op 0 of NULL. Dit is cruciaal omdat C geheugen niet automatisch wist; zonder initialisatie bevat je variabele willekeurige "garbage data".
3.  **Overerving in `fork` (`kernel/proc.c`):** Indien de data moet worden doorgegeven aan child processen, kopieer de waarde dan in de `fork()` functie.

## Context Switch Mechanism
When xv6 switches from process A to process B:
1. Process A enters the kernel via a trap.
2. `sched()` calls `swtch(&p->context, &c->context)`.
3. `swtch` saves A's registers in its `struct proc` and loads the CPU's scheduler registers.
4. The scheduler picks process B.
5. `swtch(&c->context, &p->context)` loads B's registers from its `struct proc`.
