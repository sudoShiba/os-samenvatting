# RISC-V Calling Conventions

Understanding calling conventions is essential when writing assembly code or debugging kernel-user transitions. xv6 follows the standard RISC-V calling convention.

## Registers

| Register | Name | Role | Saver |
| :--- | :--- | :--- | :--- |
| `x0` | `zero` | Always zero | - |
| `x1` | `ra` | Return address | Caller |
| `x2` | `sp` | Stack pointer | Callee |
| `x5-x7` | `t0-t2` | Temporaries | Caller |
| `x8` | `s0 / fp` | Saved register / Frame pointer | Callee |
| `x9` | `s1` | Saved register | Callee |
| `x10-x11` | `a0-a1` | Function arguments / Return values | Caller |
| `x12-x17` | `a2-a7` | Function arguments | Caller |
| `x18-x27` | `s2-s11` | Saved registers | Callee |
| `x28-x31` | `t3-t6` | Temporaries | Caller |

## Argument Passing
- The first 8 integer/pointer arguments are passed in registers **`a0`** through **`a7`**.
- Additional arguments are passed on the stack.
- Return values are placed in **`a0`** (and **`a1`** if needed).

## Stack Frame
The stack grows downwards. The stack pointer (`sp`) must always be 16-byte aligned. When a function is called, it usually creates a stack frame by decrementing `sp`, saving the return address (`ra`) and any callee-saved registers it intends to use.
