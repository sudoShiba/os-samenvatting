# Process API: fork, exec, wait, exit

The core of Unix process management. You must master these for the exam.

## 1. `fork()`
Creates a **duplicate** of the current process.
- **Return Value:** 
    - In child: `0`.
    - In parent: `PID` of the child.
    - On error: `-1`.
- **Memory:** Copies the entire address space (or uses COW).

## 2. `exec(path, argv)`
**Replaces** the current process image with a new program.
- Does **not** create a new process; the PID remains the same.
- `argv` is an array of strings, ending with a null pointer (`0`).
- **Never returns** on success.

## 3. `wait(status)`
The parent process pauses until **one** of its children terminates.
- Returns the `PID` of the terminated child.
- Collects the child's exit status.
- Prevents "Zombie" processes (processes that finished but still take up space in the `proc` table).

## 4. `exit(status)`
Terminates the current process.
- Closes all open files.
- Switches the state to `ZOMBIE`.
- Wakes up the parent so it can call `wait()`.

## The Shell Pattern
```c
int pid = fork();
if(pid == 0){
  // Child
  char *argv[] = {"ls", "-l", 0};
  exec("/bin/ls", argv);
} else {
  // Parent
  wait(0);
}
```
