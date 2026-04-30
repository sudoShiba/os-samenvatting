# Summary of Labs

This table provides a quick reference to the core concepts and primary technical focus of each lab session.

| Lab | Title | Core Concept | Primary Technical Focus |
| :--- | :--- | :--- | :--- |
| 1 | OS Interfaces | User-space API | `fork`, `exec`, `wait`, Pipes, File Descriptors |
| 2 | System Calls | Kernel Entry | Trap handling, System call dispatch, `argint`/`argaddr` |
| 3 | Context Switch | Scheduling | Process state, `swtch`, Timer interrupts, User-threads |
| 4 | Page Tables | Memory Virtualization | Page table walking, `mappages`, vDSO, Virtual Address Spaces |
| 5 | Page Faults | Lazy Management | Copy-on-Write, Reference counting, Page fault traps |
| 6 | File Systems | Storage & Abstraction | Disk layout, Inodes, Directory entries, Symbolic links |
| 7 | Advanced VM | Complex Mappings | File-backed `mmap`, COW Null Page, Lazy loading from disk |

## Progress Tracker
Each lab builds upon the previous ones. A solid understanding of address spaces (Lab 4) is critical for implementing COW (Lab 5), and both are required for advanced mappings (Lab 7). Similarly, the file system knowledge from Lab 6 is essential for file-backed mmap in Lab 7.
