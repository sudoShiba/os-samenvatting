# Key Files in xv6

This page provides an overview of the most important files in the xv6 kernel and user space, grouped by their functionality.

## Kernel Space

### Process Management
- **`kernel/proc.c`**: Contains the core logic for process management, including allocation (`allocproc`), context switching (`swtch`), and the scheduler.
- **`kernel/proc.h`**: Defines the `struct proc` (process control block) and `struct cpu`.
- **`kernel/swtch.S`**: Assembly code for saving and restoring context during a process switch.

### Memory Management
- **`kernel/vm.c`**: The virtual memory system. Includes functions for creating page tables, mapping virtual addresses to physical addresses (`mappages`), and copying address spaces (`uvmcopy`).
- **`kernel/kalloc.c`**: The physical memory allocator. Manages the free list of 4KB pages.
- **`kernel/riscv.h`**: Definitions for RISC-V specific registers, page table entry flags (PTE_V, PTE_R, PTE_W, etc.), and address translation macros.

### Traps and System Calls
- **`kernel/trap.c`**: Handles all traps (interrupts, exceptions, and system calls). Dispatches to the appropriate handler based on the `scause` register.
- **`kernel/syscall.c`**: The system call dispatcher. Maps system call numbers to their implementation functions.
- **`kernel/sysproc.c`**: Implementations for process-related system calls like `fork`, `exit`, `wait`, and `sbrk`.
- **`kernel/sysfile.c`**: Implementations for file-related system calls like `open`, `read`, `write`, and `close`.

### File System
- **`kernel/fs.c`**: The on-disk file system implementation. Manages inodes, directories, and data blocks.
- **`kernel/fs.h`**: Definitions for file system structures, including the superblock and disk inodes (`struct dinode`).
- **`kernel/file.c`**: Provides the file abstraction layer, connecting inodes/pipes to file descriptors.

## User Space
- **`user/user.h`**: The main header for user-level programs. Contains system call prototypes and library function declarations.
- **`user/usys.pl`**: A Perl script that generates the assembly stubs for system calls, which use the `ecall` instruction to enter the kernel.
- **`user/ulib.c`**: Implementation of common user-level library functions (e.g., `strcpy`, `strcmp`).

---

## Lab-Specific Files

### Lab 2: System Calls
- **`kernel/trace.c`**: Implementation of the `traceme` and `tracemefd` logic.

### Lab 3: Context Switch
- **`user/uthread.c`**: Implementation of user-level threads.
- **`user/uthread_switch.S`**: Assembly context switch for user threads.

### Lab 4: Page Tables
- **`kernel/vm.c`**: Modifications for `vmprintmappings` and `mmap`.
- **`user/vmprint.c`**: Utility to print page table mappings.

### Lab 6: File Systems
- **`user/symlinktest.c`**: Test suite for symbolic links.
