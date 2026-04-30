# Trapframe Offsets

The `struct trapframe` is a critical structure that sits at a fixed virtual address in every process's page table. It is used by the `trampoline.S` assembly code to save and restore user registers during traps.

## Struct Definition (`kernel/proc.h`)
```c
struct trapframe {
  /*   0 */ uint64 kernel_satp;   // kernel page table
  /*   8 */ uint64 kernel_sp;     // top of process's kernel stack
  /*  16 */ uint64 kernel_trap;   // usertrap()
  /*  24 */ uint64 epc;           // saved user program counter
  /*  32 */ uint64 kernel_hartid; // saved kernel tp
  /*  40 */ uint64 ra;
  /*  48 */ uint64 sp;
  /*  ... */
  /* 112 */ uint64 a0;
  /* ... */
};
```

## Usage in Assembly (`kernel/trampoline.S`)
Because assembly code cannot easily parse C structs, the offsets are hardcoded. For example, saving the user's `a0` register looks like this:
```assembly
sd a0, 112(sscratch) # Save a0 to the trapframe (sscratch points to trapframe)
```

## Key Offsets
- **0**: Kernel SATP (used to switch to the kernel page table).
- **8**: Kernel Stack Pointer.
- **16**: Address of `usertrap()`.
- **24**: User Program Counter (saved in `sepc` by hardware).
- **112**: Argument 0 (`a0`), which also holds the system call return value.

If you modify `struct trapframe`, you **must** update the corresponding offsets in `trampoline.S`.
