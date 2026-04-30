# Lab 1: OS Interfaces (Expert Depth)

The foundation of user-space programming. Focus on `fork`, `exec`, and file descriptor manipulation.

## 1. The Fork-Exec-Wait Pattern
Used to run a subprocess and wait for its completion.
```c
int pid = fork();
if(pid == 0){
  // Child: setup arguments and exec
  char *args[] = {"ls", 0};
  exec("ls", args);
  printf("exec failed\n");
  exit(1);
} else if(pid > 0){
  // Parent: wait for child
  wait(0);
}
```

## 2. Implementing `xargs` (Line-by-Line Execution)
Reading from stdin and executing a command for each line.
```c
// Inside while(readline(0, buf, sizeof(buf)) > 0)
int pid = fork();
if(pid == 0){
  // Add the line from stdin (buf) to arguments
  args[argc-1] = buf;
  args[argc] = 0;
  exec(argv[1], args);
  exit(1);
} else {
  wait(0);
}
```

## 3. Pipe Redirection
Redirecting output of one process to the input of another.
```c
int p[2];
pipe(p);
if(fork() == 0){
  close(0);     // Close stdin
  dup(p[0]);    // Duplicate pipe read end to stdin
  close(p[0]);
  close(p[1]);
  exec("wc", args);
}
```

## 4. Introspection (Memory Layout Sharing)
Comparing memory addresses and values between parent and child via pipes.
**The Pattern:**
1. Child allocates memory (`sbrk`) and sets values.
2. Child sends its local pointers (virtual addresses) and the *values* at those addresses to the parent through a `pipe`.
3. Parent reads the layout and compares it with its own.
```c
struct memlayout { void* stack; void* heap; };
struct memvalues { int stack_val; int heap_val; };

// Child side
write(pipefd[1], &layout, sizeof(layout));
write(pipefd[1], &values, sizeof(values));

// Parent side
read(pipefd[0], &child_layout, sizeof(child_layout));
printf("Child heap at %p, value %d\n", child_layout.heap, child_values.heap_val);
```
**Important:** User addresses are virtual. The parent and child may see the same virtual address, but they point to different physical frames.

