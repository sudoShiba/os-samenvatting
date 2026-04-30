# Traps, Trampoline and Trapframe

In xv6, a "trap" is any event that causes the CPU to switch from user mode to kernel mode. This includes system calls (`ecall`), hardware interrupts (timer, disk), and exceptions (page faults).

## The Trampoline Page
The **Trampoline** is a single page of memory that contains the executable code for entering and leaving the kernel.

- **Fixed Address:** It is mapped at the very top of the virtual address space (`0x3ffffff000`) in **both** the user page table and the kernel page table.
- **Why?** When switching from user to kernel mode, xv6 must change the `satp` register (which points to the page table). Because the code currently executing must remain valid during this switch, the Trampoline page is mapped at the same virtual address in both tables.
- **Source:** `kernel/trampoline.S` contains the assembly code `uservec` and `userret`.

## The Trapframe
Since the kernel needs a place to save the user's registers (so it can restore them later), every process has a dedicated **Trapframe** page.

- **Mapping:** It is mapped just below the Trampoline page in the user address space.
- **Contents:** It stores all 32 RISC-V registers, the kernel stack pointer, the address of the kernel's trap handler (`usertrap`), and the kernel page table pointer.
- **Source:** Defined as `struct trapframe` in `kernel/proc.h`.

## The Trap Flow

1.  **User Space:** A trap occurs (e.g., `ecall`). The hardware automatically:
    - Saves the current PC in `sepc`.
    - Switches to supervisor mode.
    - Jumps to the address in `stvec` (which points to `uservec` in the trampoline).
2.  **Trampoline (`uservec`):**
    - Swaps user `a0` with `sscratch` (which holds the pointer to the trapframe).
    - Saves all user registers into the trapframe.
    - Loads the kernel stack pointer and kernel page table from the trapframe.
    - Switches the `satp` register to the kernel page table.
    - Jumps to `usertrap()` in `kernel/trap.c`.
3.  **Kernel (`usertrap`):**
    - Identifies the cause of the trap (`scause`).
    - Dispatches to `syscall()`, a device driver, or handles an exception (like a page fault).
4.  **Returning (`usertrapret` -> `userret`):**
    - The reverse process occurs. `userret` switches back to the user page table, restores the user registers from the trapframe, and uses the `sret` instruction to return to user space.

## Key Files
- **`kernel/trampoline.S`**: The low-level assembly for the transition.
- **`kernel/trap.c`**: High-level C code that decides what to do with a trap.
- **`kernel/proc.h`**: Definition of the `trapframe` structure.
