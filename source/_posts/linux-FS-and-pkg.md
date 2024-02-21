---
title: linux FS and pkg
date: 2024-02-21 14:38:04
tags: Linux
category: Learning
---

# Linux filesystem and package management

## Index
1. [Filesystems](#filesystems)


## Filesystems
Some concept in filesystems are:
- Drive: A physical block device such as HDD or SSD. __In virtual machines, drive can be emulated__. <code>/dev/sda (SCSI device)</code> <code>/dev/sdb (SATA)</code> or <code>/dev/hda (IDE)</code>

- Partition: Divide physical disks into partitions, logical sub-disks

- Volume: special partition with more flexibility 
- Super block: A special section that contains metadata of the filesystem such as fs type, blocks, state..
- Inodes: Inodes store metadata about files such as size, owner, location, data and permissions. It does not store file name and actual data

Use ``` lsblk``` to list the block devices

``` bash
    lsblk --exclude 7
NAME  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda     8:0    0 389.8M  1 disk
sdb     8:16   0     3G  0 disk [SWAP]
sdc     8:32   0     1T  0 disk /var/lib/docker
                                /var/snap/firefox/common/host-hunspell
                                /snap
                                /mnt/wslg/distro
                                /
```

Use ``` findmnt``` to list filesystems
``` bash

    findmnt -D -t nosquashfs # exclude squashfs type files(used for snapshots)
```

### Hard link and symbolic links

 A hard link works by creating another filename that refers to the inode data of the original file. In practice, this is similar to creating a copy of the file.

- If the original file is deleted, the file data can still be accessed through other hard links.

- If the original file is moved, hard links still work.

-  A hard link can only refer to a file on the same file system.

- The inode and file data are permanently deleted when the number of hard links is zero

``` bash
    ln myfile somealias
```
A soft link, sometimes called a symbolic link or symlink, points to the location or path of the original file. It works like a hyperlink on the internet.


-  If the symbolic link file is deleted, the original data remains.

-  If the original file is moved or deleted, the symbolic link won’t work.

-  A soft link can refer to a file on a different file system.

-  Soft links are often used to quickly access a frequently-used file without typing the whole location.

``` bash
    ln -s myfile somealias
```
Differences between hard link and symbolic link

- A soft link does not contain the data in the target file.

- A soft link points to another entry somewhere in the file system.

- A soft link has the ability to link to directories, or to files on remote computers networked through NFS.

- Deleting a target file for a symbolic link makes that link useless.

- A hard link preserves the contents of the file.

- A hard link cannot be created for directories, and they cannot cross filesystem boundaries or span across partitions.

- In a hard link, you can use any of the hard link names created to execute a program or script in the same manner as the original name given.


### VFS (virtual file system)
Abstraction layer in the kernel that provides clients a common way to access resources.
Using VFS user can access to resource like local filesystems, in-memory filesystems(memory), pseudo filesystems or network filesystems and treat them as simple file.
[https://www.kernel.org/doc/html/next/filesystems/vfs.html]