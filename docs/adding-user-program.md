# Adding a User Program

To add a new user-space program to xv6:

## 1. Create the Source File
Create your C file in the `user/` directory (e.g., **`user/hello.c`**).
```c
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
  printf("Hello from user space!\n");
  exit(0);
}
```

## 2. Update the Makefile
Add your program to the `UPROGS` list in the **`Makefile`**.
```makefile
UPROGS=\
	$U/_cat\
	$U/_echo\
	$U/_hello\
    ...
```
Note the underscore (`_`) prefix in the Makefile entry.

## 3. Build and Run
Run `make qemu` to compile your program and include it in the file system image. You can then run it from the xv6 shell:
```bash
$ hello
Hello from user space!
```
