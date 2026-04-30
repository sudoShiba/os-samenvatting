# Adding a User Library Function

Sometimes you want to add a function that is available to all user programs (like `printf` or `strcpy`), but isn't a direct system call.

## 1. Header Declaration (`user/user.h`)
Add the prototype to the end of the "library functions" section.
```c
char* gets(char*, int);
uint strlen(const char*);
void* memset(void*, int, uint);
// Your new function:
int my_helper(int); 
```

## 2. Implementation (`user/ulib.c`)
Most user library functions live in `user/ulib.c`.
```c
int
my_helper(int x)
{
  // Implementation logic here
  // You can use other syscalls like write() or read()
  return x * 2;
}
```

## 3. Alternative: New Library File
If you are adding a large library (like `uthread` in Lab 3):
1. Create `user/mylib.c`.
2. Update the **`Makefile`** to include it in the `ULIB` variable:
   ```makefile
   ULIB = $U/ulib.o $U/usys.o $U/printf.o $U/umalloc.o $U/mylib.o
   ```

## Key Difference: Syscall vs Library
- **Syscall:** Requires `ecall` to switch to kernel mode. Defined in `usys.pl`.
- **Library Function:** Runs entirely in user mode. Just a standard C function called by other programs.
