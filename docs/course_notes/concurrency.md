# Concurrency

Concurrency involves executing multiple instruction streams at the same time, often within a shared memory space.

## 1. Threads
- A **thread** is an abstraction of a single sequential control flow within a process.
- Multiple threads within the same process share the same virtual address space (code, heap, globals) but have their own private execution context (Registers, Program Counter, Stack).
- **Why threads?**
  1. Utilize multiple physical CPUs (Parallelism).
  2. Overlap I/O and computation (if one thread blocks on I/O, another can run).

### Thread API (POSIX)
- `pthread_create()`: Spawn a new thread.
- `pthread_join()`: Wait for a thread to finish.

## 2. Synchronization Problems
Unexpected interleavings of concurrent threads can cause errors, primarily **Data Races** and **Race Conditions**.
- e.g., Two threads incrementing a shared counter `counter = counter + 1` at the same time. The assembly instructions (load, add, store) can be interleaved, leading to lost updates.

## 3. Synchronization Primitives

To solve concurrency issues, the OS (or language runtime) provides synchronization primitives:

### Locks (Mutexes)
- Ensure the **atomicity** of critical sections.
- Only one thread can hold the lock at a time.
- `pthread_mutex_lock()` and `pthread_mutex_unlock()`.

### Condition Variables
- Enable threads to **wait** for specific events or conditions to become true, without busy-waiting (spinning).
- `pthread_cond_wait(&cond, &lock)`: Atomically releases the lock and puts the thread to sleep. When woken up, it re-acquires the lock.
- `pthread_cond_signal(&cond)`: Wakes up one waiting thread.
- **Always use a `while` loop around `cond_wait`**, because of spurious wakeups or because the condition might have changed between the wakeup and the thread acquiring the lock.

## 4. Concurrency Bugs
- **Deadlocks:** Two or more threads are blocked forever, waiting for each other.
- **Data Races:** Concurrent access to shared data where at least one access is a write, without proper synchronization.
- Tools like `tsan` (ThreadSanitizer) help detect these issues.
