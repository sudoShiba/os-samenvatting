# Persistence

Persistence deals with how information can be stored reliably across system restarts and crashes. This is primarily handled by the **File System**.

## 1. File System Interface

The OS abstracts low-level device interactions (like reading/writing disk sectors) into a higher-level abstraction: the File System.

### Key Abstractions
- **File:** A persistent array of bytes.
- **Directory:** A listing that maps human-readable file names to low-level on-disk indices ("inodes"). Directories form a hierarchical tree.

### Identifying Files (The Trinity)
A file can be identified in three distinct ways:
1. **System Level (Inode):** The unique on-disk index (e.g., `177516`).
2. **Human Level (Path):** The human-readable hierarchical name (e.g., `/tmp/foo`).
3. **Process Level (File Descriptor):** An integer returned by `open()` (e.g., `3`). This is an index into a per-process open-file table, which points to a system-wide open-file table.
   - **Why File Descriptors?** They speed up path traversals for repeated access and allow processes to share the same file offset or redirect I/O.

![File System Navigation](https://imgs.xkcd.com/comics/file_system.png)
*(Navigating the file system abyss, XKCD 1360)*

### Standard Unix I/O
- Every process starts with 3 open files: `stdin` (0), `stdout` (1), and `stderr` (2).
- The "Everything is a File" philosophy means even devices and network sockets can be accessed via the standard file API (`read()`, `write()`, `close()`).

---

## 2. File System Implementation

How do we organize data and metadata on the raw disk blocks?

### On-Disk Data Structures
- **Superblock:** Essential information about the file system (e.g., number of inodes, data blocks, magic number).
- **Bitmaps (Free Space Management):** Track which inodes and data blocks are currently allocated or free.
  - **Data Bitmap:** 1 bit per data block.
  - **Inode Bitmap:** 1 bit per inode.
- **Inode Table:** An array of inodes on disk.
- **Data Region:** The actual blocks storing user data.

### The Ext2 Inode Layout
In place of a broken image, here is the structure of an Ext2-style inode:

| Field | Description |
| :--- | :--- |
| **Mode** | File type and access permissions (read/write/exec). |
| **Owner Info** | UID/GID of the file owner. |
| **Size** | Size of the file in bytes. |
| **Timestamps** | Atime, Mtime, Ctime (Access, Modification, Change). |
| **Links Count** | Number of hard links pointing to this inode. |
| **Blocks Count** | Number of disk blocks used by the file. |
| **Direct Blocks** | 12 pointers directly to data blocks. |
| **Single Indirect** | Pointer to a block containing more data block pointers. |
| **Double Indirect** | Pointer to a block containing pointers to indirect blocks. |
| **Triple Indirect** | Pointer to a block containing pointers to double indirect blocks. |

*(Reference: OSTEP File System Implementation)*

### The Inode (Index Node)
The inode stores all **metadata** about a file:
- **Type:** File, Directory, Symlink, Device.
- **Size:** Size in bytes.
- **Permissions/Mode:** Read/Write/Execute bits.
- **Pointers to Data Blocks:** How to find the file's data on disk.
  - **Multi-Level Indexing:** To support large files, an inode typically has a fixed number of *direct pointers*, plus a *single indirect pointer*, a *double indirect pointer*, etc.

### Directories
- A directory is just a file where the data blocks contain a list of `(filename, inode_number)` mappings.
- Searching for a file (e.g., `/foo/bar`) requires traversing these directory data blocks to find the corresponding inode number.

---

## 3. Crash Consistency

What happens if the system crashes (e.g., power loss) during a non-atomic file system update? For example, appending to a file requires updating the data bitmap, the inode, and writing the actual data block. If only the data bitmap is written before a crash, we have a "space leak".

### Solution 1: File System Checker (`fsck`)
- **Lazy Check and Repair:** Let things go wrong, then find and fix inconsistencies later (on reboot) by scanning the entire disk.
- **How?** Exploits inherent redundancy in file-system metadata (e.g., comparing directory entries with inode link counts, or used blocks with the data bitmap).
- **Problem:** Full disk scan is incredibly **slow** on modern, large disks.

### Solution 2: Journaling (Write-Ahead Logging)
- **Idea:** Before overwriting the actual disk structures, first write a brief note (the "journal") describing the intended update.
- **Protocol (Metadata Journaling):**
  1. **Data write:** Write data to final location.
  2. **Journal metadata write:** Write the begin block and metadata to the log.
  3. **Journal commit:** Write the transaction commit block to the log. (Transaction is now committed).
  4. **Checkpoint:** Write the metadata to its final location.
  5. **Free:** Mark the transaction as free in the journal superblock.
- **Advantage:** If a crash occurs, the OS only needs to replay the journal, not scan the entire disk. Fast recovery!

![Abstractions](https://imgs.xkcd.com/comics/abstractions.png)
*(The power of abstractions, XKCD 676)*
