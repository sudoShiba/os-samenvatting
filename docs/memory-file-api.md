# Memory & File API

## Memory: `sbrk(n)`
Increases or decreases the process's data space (heap) by `n` bytes.
- Returns the **previous** end of the data space.
- The kernel handles this by modifying `p->sz` and mapping/unmapping pages.

## Files: `pipe(p)`
Creates a unidirectional data channel.
- `p[0]` is the read end.
- `p[1]` is the write end.
- Often used with `fork()` and `dup()` for redirection.

## Files: `dup(fd)`
Duplicates an existing file descriptor.
- The new FD points to the **same** underlying file object as `fd`.
- Often used to redirect stdin (0) or stdout (1).
```c
close(0);
dup(p[0]); // p[0] is now stdin
```
