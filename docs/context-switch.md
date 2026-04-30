# Lab 3: Context Switch (Expert Depth)

Understanding how xv6 manages CPU state and switches between processes is vital.

## 1. Kernel Context Switch (`swtch.S`)
Registers saved during a kernel-level switch.
```assembly
# void swtch(struct context *old, struct context *new);
.globl swtch
swtch:
  sd ra, 0(a0)
  sd sp, 8(a0)
  sd s0, 16(a0)
  # ... save s1-s11 ...
  
  ld ra, 0(a1)
  ld sp, 8(a1)
  ld s0, 16(a1)
  # ... restore s1-s11 ...
  ret
```

## 2. User-level Threads (`uthread`)
Implementing threading in user-space without kernel help.

**Thread Context (`user/uthread.h`):**
```c
struct thread_context {
  uint64 ra;
  uint64 sp;
  uint64 s0;
  uint64 s1;
  // ... s2-s11 ...
};
```

**Thread Creation (`user/uthread.c`):**
```c
struct thread *thread_create(void (*func)()) {
  struct thread *t = find_unused_thread();
  // Set return address to the function entry point
  t->context.ra = (uint64)func;
  // Set stack pointer to top of the thread's stack
  t->context.sp = (uint64)t->stack + STACK_SIZE;
  t->state = RUNNABLE;
  return t;
}
```

**Thread Switcher (`user/uthread_switch.S`):**
Identical logic to `swtch.S` but works on `struct thread_context` in user memory.

## 3. Trap Handling (`kernel/trap.c`)
During the exam, you may need to improve kernel diagnostics.
**Solution Pattern for `usertrap()`:**
```c
  } else {
    // Check for specific exceptions
    if (scause == 15) { // Store Page Fault
      uint64 vma = r_stval();
      if (vma == 0) {
        printf("usertrap(): store to nullptr (0x%lx) pid=%d\n", scause, p->pid);
        setkilled(p);
      }
    } else if (scause == 2) { // Illegal Instruction
      printf("usertrap(): illegal instruction (0x%lx) pid=%d\n", scause, p->pid);
      setkilled(p);
    }
  }
```

## 4. Interrupt Counters
Tracking hardware events in the kernel.
**Structure in `kernel/proc.h`:**
```c
struct interrupt_c {
  uint64 timer;
  uint64 external;
  uint64 software;
};

struct cpu {
  // ... existing fields ...
  struct interrupt_c interrupts;
};
```
**Updating in `kernel/trap.c`:**
In `devintr()`, increment the counter for the current CPU: `cpus[cpuid()].interrupts.timer++;`.

