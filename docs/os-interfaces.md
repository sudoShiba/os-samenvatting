# Lab 1: OS Interfaces (Expert Depth)

The foundation of user-space programming. Focus on `fork`, `exec`, and file descriptor manipulation.

## 1. The Fork-Exec-Wait Pattern
Used to run a subprocess and wait for its completion.
```c
int pid = fork();
if(pid < 0){
  printf("fork failed\n");
  exit(1);
} else if(pid == 0){
  // Child: setup arguments and exec
  char *args[] = {"ls", 0};
  exec("ls", args);
  printf("exec failed\n");
  exit(1);
} else {
  // Parent: wait for child
  wait(0);
}
```

## 2. Implementing `xargs` (Line-by-Line Execution)
Reading from stdin and executing a command for each line.

**Helper: `readline`**
Reads one line from a file descriptor into a buffer.
```c
int readline(int fd, char *buf, int size) {
  char c;
  int i = 0;
  while ((i < size - 1) && read(fd, &c, 1) > 0) {
    if (c == '\n') {
      buf[i] = '\0';
      return i + 1;
    }
    buf[i++] = c;
  }
  buf[i] = '\0';
  return i;
}
```

**Main Logic:**
```c
// Inside while(readline(0, buf, sizeof(buf)) > 0)
int pid = fork();
if (pid > 0) {
  /* parent */
  wait(0);
} else if (pid == 0) {
  /* child */
  args[argc-1] = buf;
  args[argc] = 0;
  exec(argv[1], args);
  exit(1);
} else {
  printf("fork error\n");
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
Comparing memory addresses and values between parent and child via pipes. In xv6, virtual addresses might be the same, but they point to different physical memory.

**Implementation Logic:**
1. Child and parent both track a `struct memlayout` (pointers to stack/heap/data).
2. Child sends its `layout` and `values` to the parent via a pipe.
3. Parent reads these and prints both sets to compare.

```c
if (fork_result == 0) {
  // Child: modify local variables and update layout
  stack_var = 1;
  data_var = 1;
  // ...
  write(pipefd[1], &layout, sizeof(layout));
  write(pipefd[1], &values, sizeof(values));
  close(pipefd[1]);
} else {
  // Parent: read child's data
  read(pipefd[0], &child_layout, sizeof(child_layout));
  read(pipefd[0], &child_values, sizeof(child_values));
  // Compare with local parent layout
}
```
**Key Insight:** User addresses are virtual. If `&stack_var` is `0x3fffe0` in both, they represent different physical pages in the respective page tables.
