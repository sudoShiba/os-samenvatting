# Virtualization

Virtualization is the process of creating a virtual (rather than actual) version of something, such as a hardware platform, storage device, or network resources. In the context of Operating Systems, it primarily refers to **CPU Virtualization** and **Memory Virtualization**.

## 1. CPU Virtualization (Processes and Scheduling)

The OS provides the illusion of an unbounded number of (virtual) CPUs, even if there is only a small number of physical CPUs. The key abstraction is the **Process** ("a running program").

### The Process Abstraction
- A process encapsulates all the state of a "running program", including memory and register contents.
- **Process States:** Running, Ready, Blocked (e.g., waiting for I/O).
- **Process List:** A data structure maintained by the OS containing Process Control Blocks (PCBs) for each process.
- **Unix Process API:**
  - `fork()`: Creates a new process by duplicating the caller.
  - `wait()`: Waits on the completion of another process.
  - `exec()`: Transforms the calling process to run another program.

### Limited Direct Execution
To virtualize the CPU efficiently without losing control:
- **Direct Execution:** Code runs directly on the hardware.
- **Limited:** We restrict dangerous operations using **CPU modes** (user mode vs. kernel mode).
- **Traps:** To do something privileged, a process must execute a system call (`ecall`), which traps into the OS (kernel mode).
- **Regaining Control:** The OS uses a timer interrupt to forcefully regain control from a running process and perform a context switch.

### Scheduling
The OS needs a policy to decide which process gets the CPU.
- **Metrics:** Turnaround time (performance) vs. Fairness/Response time.
- **Policies:**
  - **FIFO (First In, First Out):** Simple but suffers from the convoy effect.
  - **SJF (Shortest Job First):** Optimal for turnaround time if all jobs arrive at once, but bad for late arrivals.
  - **STCF (Shortest Time-to-Completion First):** Preemptive SJF.
  - **RR (Round Robin):** Switches fast between jobs. Good for response time, bad for turnaround time.
  - **MLFQ (Multi-Level Feedback Queue):** Learns from history. High-priority queues for interactive (I/O) jobs, low-priority for CPU-bound jobs. Uses priority boosts to prevent starvation.
  - **Lottery/Stride Scheduling:** Proportional share schedulers using randomness or deterministic strides.
  - **Linux CFS (Completely Fair Scheduler):** Uses virtual runtime and a balanced tree to fairly divide CPU time.

---

## 2. Memory Virtualization (Address Spaces)

The OS provides the illusion of a private, large address space for multiple processes on top of a single physical memory.

### The Address Space Abstraction
- Programs run in an abstract machine with a private virtual address space starting at address zero.
- **Segmentation:** Divide the address space into segments (Code, Heap, Stack). Each segment has a base/limit pair. Causes **External Fragmentation**.
- **Paging:** Chop the address space into fixed-sized units called **pages** (virtual) and **frames** (physical). No external fragmentation.

### Address Translation and Page Tables
- The MMU (Memory Management Unit) hardware translates virtual addresses to physical addresses using a **Page Table** filled by the OS.
- **Linear Page Table:** An array of Page Table Entries (PTEs). Indexed by the Virtual Page Number (VPN).
- **PTE:** Contains the Physical Page Number (PPN) and flags (Valid, Read, Write, Execute, User, Dirty, Accessed).
- **Problem:** Page tables can be huge.
- **Solution: Multi-Level Page Tables:** Chop the page table into page-sized parts. Only allocate parts that are in use. Forms a search tree (e.g., Page Directory -> Page Table -> Physical Page).

### Speeding up Translation: TLB
- **TLB (Translation Lookaside Buffer):** A small, fast hardware cache in the MMU that stores recent virtual-to-physical translations.
- Relies on spatial and temporal locality.

### Swapping (Demand Paging)
How to create the illusion of unlimited memory?
- Store unused memory pages on the disk (**Swap Space**).
- Hardware generates a **page fault** when a virtual page is not present in RAM (Valid bit = 0).
- OS page-fault handler swaps the page back in.
- **Page Replacement Policies:** When RAM is full, which page to evict?
  - **FIFO / Random:** Simple but often suboptimal.
  - **LRU (Least Recently Used):** Good heuristic based on history.
  - **Clock Algorithm:** An efficient approximation of LRU using an "Accessed" bit set by the hardware.

![Compiler Compiler](https://imgs.xkcd.com/comics/compiler_complaint.png)
*(A classic struggle with pointers and segfaults, courtesy of XKCD 371)*
