# Lab 3: Context Switch (Expert Depth)

Understanding how xv6 manages CPU state and switches between processes is vital.

## 1. Kernel Context Switch (`swtch.S`)
Registers saved during a kernel-level switch. callee-saved registers (s0-s11), ra, and sp must be preserved.
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
Implementing threading in user-space. The logic for switching is identical to the kernel `swtch`, but applied to user-space thread structures.

**Thread Creation:**
```c
struct thread *t = find_unused_thread();
t->context.ra = (uint64)func;
t->context.sp = (uint64)t->stack + STACK_SIZE;
t->state = RUNNABLE;
```

## 3. Trap Handling & Diagnostics (`kernel/trap.c`)
Improving kernel diagnostics for common user errors like null pointer dereferences.

**Pattern in `usertrap()`:**
```c
  } else if((r_scause() == 15 || r_scause() == 13) &&
            vmfault(p->pagetable, r_stval(), (r_scause() == 13)? 1 : 0) != 0) {
    // page fault on lazily-allocated page
  } else {
    uint64 scause = r_scause();
    if (scause == 15) {
      uint64 vma = r_stval();
      if (vma == 0) {
        printf("usertrap(): store to nullptr (0x%lx)  pid=%d\n", scause, p->pid);
        printf("            at program counter (pc) = 0x%lx\n", r_sepc());
        setkilled(p);
      }
    } else if (scause == 2) {
      printf("usertrap(): illegal instruction (0x%lx) pid=%d\n", scause, p->pid);
      setkilled(p);
    } else {
      printf("usertrap(): unexpected scause 0x%lx pid=%d\n", scause, p->pid);
      setkilled(p);
    }
  }
```

## 4. Interrupt Counters
Tracking hardware events per CPU.

**Structure in `kernel/proc.h`:**
```c
struct interrupt_c {
  uint64 timer;
  uint64 uart;
  uint64 disk;
};

struct cpu {
  struct proc *proc;
  struct context context;
  int noff;
  int intena;
  struct interrupt_c interrupts; // Counter structure
  int init;
};
```

**Updating in `kernel/trap.c`:**
In `devintr()`, use `mycpu()` to increment the counter:
```c
  if(scause == 0x8000000000000009L){
    int irq = plic_claim();
    if(irq == UART0_IRQ){
      mycpu()->interrupts.uart++;
      uartintr();
    } else if(irq == VIRTIO0_IRQ){
      mycpu()->interrupts.disk++;
      virtio_disk_intr();
    }
    // ...
  } else if(scause == 0x8000000000000005L){
    mycpu()->interrupts.timer++;
    clockintr();
  }
```
**Exam Note:** If asked to implement `sigalarm`, you would typically add `alarm_interval`, `alarm_handler`, and `ticks_count` to `struct proc`, then check `ticks_count` in `usertrap` during a timer interrupt.
