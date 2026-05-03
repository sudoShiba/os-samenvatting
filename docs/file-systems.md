# Lab 6: File Systems (Expert Depth)

De disk in xv6 is opgedeeld in blokken van **1024 bytes** (BSIZE).

## 0. Disk Layout & Inode Limits
De disklayout bepaalt waar welk soort data staat:
`[ boot | super | log | inodes | bit map | data ... ]`
- **boot:** Bevat de boot code (blok 0).
- **super:** Bevat metainformatie over het bestandssysteem.
- **log:** Transacties voor consistentie bij crashes.
- **inodes:** Bevat alle kenmerken (type, grootte, locatie) van bestanden.
- **bitmap:** Houdt bij welke datablokken bezet zijn.
- **data:** De eigenlijke inhoud van de bestanden.

### Inode Blokken & Bestandsgrootte (Lab 2025 Config)
Een `dinode` (on-disk inode) in deze versie heeft:
- **11 directe blokken (`NDIRECT`):** Deze wijzen rechtstreeks naar datablokken.
- **1 indirect blok:** Dit wijst naar een blok dat zelf weer wijst naar datablokken.
    - Een indirect blok kan `BSIZE / sizeof(uint)` = `1024 / 4` = **256** bloknummers bevatten.
- **Totaal aantal blokken per bestand:** 11 (direct) + 256 (indirect) = **267 blokken**.
- **Maximale bestandsgrootte:** 267 * 1024 bytes ≈ **267 KB**.

### Directory Limits
Een directory entry (`struct dirent`) is **16 bytes** groot (2 bytes voor inum en 14 bytes voor de naam).
- **Entries per blok:** `1024 / 16` = **64 entries**.
- **Maximaal aantal entries per directory:** 267 blokken * 64 entries/blok = **17088 entries**.

## 1. Implementing `symlink`
A symlink is an inode of type `T_SYMLINK` that stores the target path in its data blocks.

**`sys_symlink` implementation:**
```c
uint64
sys_symlink(void)
{
  char target[MAXPATH], path[MAXPATH];
  struct inode *ip;

  if(argstr(0, target, MAXPATH) < 0 || argstr(1, path, MAXPATH) < 0)
    return -1;

  begin_op();
  if((ip = create(path, T_SYMLINK, 0, 0)) == 0){
    end_op();
    return -1;
  }

  int size = strlen(target);
  if (writei(ip, 0, (uint64) target, 0, size) < size) {
    iunlockput(ip);
    end_op();
    return -1;
  }

  iunlockput(ip);
  end_op();
  return 0;
}
```

## 2. Following Symlinks in `sys_open`
Resolution loop in `sys_open` with a depth limit to prevent infinite recursion.

```c
  } else {
    if((ip = namei(path)) == 0){
      end_op();
      return -1;
    }

    // Follow symbolic links for up to 10 links
    for (int i = 0; i < 10; ++i) {
      ilock(ip);
      if(ip->type != T_SYMLINK || omode & O_NOFOLLOW) {  goto found; }

      if (ip->type == T_SYMLINK && readi(ip, 0, (uint64) symlinktarget, 0, MAXPATH) == 0) {
        iunlockput(ip);
        end_op();
        return -1;
      }
      iunlockput(ip);
      if((ip = namei(symlinktarget)) == 0){
        end_op();
        return -1;
      }
    }
    end_op();
    return -1;

found:
    // ... check permissions ...
```

## 3. File Permissions (`chmod`)
Implementing access control via a `mode` field in both `inode` (memory) and `dinode` (disk).

**Permission Check during `open`:**
```c
  int needs_read  = !(omode & O_WRONLY);
  int needs_write = (omode & O_WRONLY) || (omode & O_RDWR);

  if ((needs_read  && !(ip->mode & M_READ)) ||
      (needs_write && !(ip->mode & M_WRITE))) {
    iunlockput(ip);
    end_op();
    return -1;
  }
```

**Syncing in `kernel/fs.c`:**
In `iupdate` and `ilock`, ensure `ip->mode` and `dip->mode` are synchronized alongside other metadata.

## 4. Large Files (Exam Variation)
If asked to implement **double indirect blocks**:
1. Change `NDIRECT` to 10.
2. `addrs[11]` becomes the single indirect block.
3. `addrs[12]` becomes the double indirect block.
4. Update `bmap` to handle the two-level jump:
```c
// Double indirect
if(bn < NINDIRECT * NINDIRECT){
  if((addr = ip->addrs[NDIRECT+1]) == 0) ... // allocate block
  bp = bread(ip->dev, addr);
  a = (uint*)bp->data;
  if((addr = a[bn / NINDIRECT]) == 0) ... // allocate intermediate block
  // ... read second level ...
}
```
