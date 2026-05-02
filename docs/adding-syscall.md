# Adding a System Call

Een system call vormt de brug tussen User Space en Kernel Space via de `ecall` instructie.

## 0. De System Call: Stap voor Stap
1.  **C-Wrapper (`user/user.h` & `user/usys.S`):** Een programma roept de functie aan. De wrapper (gegenereerd door `usys.pl`) zet het unieke syscall-nummer in register **`a7`** en de argumenten in **`a0`** tot **`a5`**.
2.  **De Trap (`ecall`):** Deze RISC-V instructie pauzeert de user mode en geeft de controle aan de kernel.
3.  **De Trampoline (`kernel/trampoline.S`):** Slaat alle user-registers op in het **trapframe** van het proces en wisselt naar de **kernel stack**.
4.  **De Dispatcher (`kernel/syscall.c`):** De kernel kijkt in register `a7`, gebruikt dit als index voor de `syscalls[]` tabel, en roept de bijbehorende C-functie aan (bijv. `sys_my_syscall`).
5.  **De Uitvoering:** De kernel-functie haalt argumenten veilig op via `argint`, `argaddr`, etc., en voert het zware werk uit. Het resultaat wordt in het **`a0`** register van het trapframe geplaatst.
6.  **De Terugkeer (`sret`):** Via `usertrapret` en de trampoline worden de user-registers hersteld en keert de CPU terug naar user mode, exact na de originele `ecall`.

## 1. User-Space Header
Add the system call prototype to **`user/user.h`**.
```c
int my_syscall(int);
```

## 2. User-Space Stub
Add an entry to **`user/usys.pl`**. This script generates the assembly code that uses the `ecall` instruction to enter the kernel.
```perl
entry("my_syscall");
```

## 3. System Call Number
Define a new system call number in **`kernel/syscall.h`**.
```c
#define SYS_my_syscall 22
```

## 4. Kernel-Space Dispatch
Update **`kernel/syscall.c`** to include the new system call in the dispatch table.
```c
// Add extern declaration
extern uint64 sys_my_syscall(void);

// Add to the syscalls array
[SYS_my_syscall] sys_my_syscall,
```

## 5. Kernel Implementation
Implement the system call logic. Most process-related calls go in **`kernel/sysproc.c`**, and file-related calls go in **`kernel/sysfile.c`**.
```c
uint64
sys_my_syscall(void)
{
  int n;
  argint(0, &n); // Retrieve the first integer argument
  // Your logic here
  return 0;
}
```
