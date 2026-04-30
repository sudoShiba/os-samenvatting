# Lab 6: File Systems (Expert Depth)

File system tasks require manipulating inodes and handling path lookup.

## 1. Implementing `symlink`
A symlink is an inode of type `T_SYMLINK` that stores the target path in its data blocks.

**`sys_symlink` implementation:**
```c
uint64 sys_symlink(void) {
  char target[MAXPATH], path[MAXPATH];
  struct inode *ip;

  if(argstr(0, target, MAXPATH) < 0 || argstr(1, path, MAXPATH) < 0)
    return -1;

  begin_op();
  if((ip = create(path, T_SYMLINK, 0, 0)) == 0){ // Create new inode
    end_op(); return -1;
  }

  int size = strlen(target);
  if (writei(ip, 0, (uint64) target, 0, size) < size) { // Write path to disk
    iunlockput(ip); end_op(); return -1;
  }

  iunlockput(ip);
  end_op();
  return 0;
}
```

## 2. Following Symlinks in `sys_open`
When opening a file, if it's a symlink, you must resolve it (unless `O_NOFOLLOW` is set).

**Updated `sys_open` loop:**
```c
// ... inside sys_open ...
if(!(omode & O_CREATE)){
  if((ip = namei(path)) == 0){
    end_op(); return -1;
  }

  // Follow symbolic links for up to 10 links
  for (int i = 0; i < 10; ++i) {
    ilock(ip);
    if(ip->type != T_SYMLINK || omode & O_NOFOLLOW) { goto found; }

    char symlinktarget[MAXPATH];
    if (readi(ip, 0, (uint64) symlinktarget, 0, MAXPATH) == 0) {
      iunlockput(ip); end_op(); return -1;
    }
    iunlockput(ip);
    if((ip = namei(symlinktarget)) == 0){
      end_op(); return -1;
    }
  }
  end_op(); return -1; // Too many links (loop)
}
found:
// ... check permissions and finish open ...
```

## 3. File Permissions (`chmod`)
Adding a `mode` bitmask to inodes.
- **In-memory inode (`kernel/file.h`)**: Add `ushort mode;`.
- **On-disk dinode (`kernel/fs.h`)**: Add `ushort mode;`.
- **Permission Check**:
```c
int needs_read  = !(omode & O_WRONLY);
int needs_write = (omode & O_WRONLY) || (omode & O_RDWR);

if ((needs_read  && !(ip->mode & M_READ)) ||
    (needs_write && !(ip->mode & M_WRITE))) {
  iunlockput(ip); end_op(); return -1;
}
```
**Exam Tip:** You must sync the `mode` field between the on-disk `dinode` and in-memory `inode` in **`kernel/fs.c`**:

**In `iupdate()` (Memory to Disk):**
```c
  dip = (struct dinode*)bp->data + ip->inum%IPB;
  dip->type = ip->type;
  dip->mode = ip->mode; // Sync mode
  memmove(dip->addrs, ip->addrs, sizeof(ip->addrs));
```

**In `ilock()` (Disk to Memory):**
```c
  if(ip->valid == 0){
    dip = (struct dinode*)bp->data + ip->inum%IPB;
    ip->type = dip->type;
    ip->mode = dip->mode; // Sync mode
    memmove(ip->addrs, dip->addrs, sizeof(ip->addrs));
    ip->valid = 1;
  }
```
