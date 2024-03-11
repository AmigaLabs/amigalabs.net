---
title: open() and opendir() functions
---

## Using open() with directories

A new feature on clib4 is the possibility to open() a directory.
Of course you can't read() or write() on the directory but you can access the directory using fstat() function for example.

New flags are now supported by open:

### O_DIRECTORY

If pathname is not a directory, cause the open to fail.

### O_PATH

Obtain a file descriptor that can be used for two purposes: 
* to indicate a location in the filesystem tree and to perform operations that act purely at the file descriptor level.  The file itself is not opened, and other file operations (e.g., `read` or `write`),  fail with the error EBADF.

The following operations can be performed on the resulting file descriptor:

  *  `close`

  *  `fchdir` if the file descriptor refers to a directory
 
  *  `fstat`

  *  `fstatfs`

  *  Duplicating the file descriptor `dup`, `fcntl`(**F_DUPFD**, etc.)

  *  Getting and setting file descriptor flags `fcntl`(**F_GETFD** and **F_SETFD**)

  *  Retrieving open file status flags using the `fcntl `with **F_GETFL** operation: the returned flags will include the bit `O_PATH`.

  *  Passing the file descriptor as the dirfd argument of `openat()` and the other "`at*()` system calls.