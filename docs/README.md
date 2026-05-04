# 🛠️ xv6 Lab Documentation

Welkom bij de centrale hub voor de xv6 labs (Besturingssystemen). Deze documentatie is ontworpen om je te helpen bij het begrijpen, implementeren en studeren van de verschillende onderdelen van de xv6 kernel.

---

## 🚀 Aan de Slag

Nieuw met xv6 of een specifieke taak aan het uitvoeren? Begin hier:

*   **[Boilerplate & Templates](boilerplate-templates.md)**: Kant-en-klare code snippets voor snelle implementatie.
*   **[Locking Guide](locking-guide.md)**: Essentieel voor Lab 6 & 7 en het begrijpen van concurrency.
*   **[Syscall Toevoegen](adding-syscall.md)**: Stap-voor-stap gids voor je eerste kernel uitbreiding.

---

## 📚 Lab Overzicht

Elk lab focust op een specifiek subsysteem van het besturingssysteem.

| Lab | Onderwerp | Kernfocus |
| :--- | :--- | :--- |
| **[Lab 1](os-interfaces.md)** | OS Interfaces | User-space API, Pipes, Fork/Exec |
| **[Lab 2](syscalls-lab.md)** | System Calls | Kernel entry/exit, argumenten ophalen |
| **[Lab 3](context-switch.md)** | Context Switch | Scheduling, process states, interrupts |
| **[Lab 4](page-tables.md)** | Page Tables | Virtueel geheugen, address translation |
| **[Lab 5](page-faults.md)** | Page Faults | Copy-on-Write, Lazy allocation |
| **[Lab 6](file-systems.md)** | File Systems | Inodes, disk layout, logging |
| **[Lab 7](advanced-vm.md)** | Advanced VM | mmap, complex memory mappings |

---

## 🎓 Examen & Theorie

Bereid je voor op het examen met deze samenvattingen en gidsen:

*   **[Lab Samenvatting](summary.md)**: Een overzicht van alle technische doelen per lab.
*   **[Decision Guide](decision-guide.md)**: Wanneer gebruik je welke kernel functie?
*   **[Common Pitfalls](pitfalls.md)**: Vermijd de meest gemaakte fouten tijdens de labs.
*   **[Exam Strategy](exam-strategy.md)**: Tips voor het aanpakken van de examenopdrachten.

---

## 🔍 Snelzoeken

Zoek je een specifieke functie of bestand?
*   [Kernel Helpers](kernel-helpers.md): `copyin`, `copyout`, `argint`, etc.
*   [Key Files](key-files.md): Wat doet `proc.c`, `fs.c`, of `vm.c` precies?
*   [API Referentie](process-api.md): Overzicht van syscalls en kernel interfaces.

---
*Laatst bijgewerkt: Mei 2026*
