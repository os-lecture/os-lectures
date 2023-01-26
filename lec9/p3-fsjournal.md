---
marp: true
theme: default
paginate: true
_paginate: false
header: ''
footer: ''
backgroundColor: white
---

<!-- theme: gaia -->
<!-- _class: lead -->

# Lecture 9 File System

## Section 3 A file system that supports crash consistency


<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Crash consistency problem
- Crash consistency
- Crash scene
2. The file system checker fsck
3. Journaling file system

---

#### Persistent Data Update Challenges for Filesystems

How to update persistent data structures in the event of a power loss or system crash.

Crashes can cause **inconsistencies** in the filesystem data structures in the disk filesystem image. For example, there is a space leak, garbage data is returned to the user, and so on.

---

#### Crash Consistency Issue

**Crash consistency** problem (crash-consistency problem) is also called **consistent update** problem (consistent-update problem)
- Certain operations require updating **two structures** A and B on disk.
- The disk only serves one request at a time, so one of the requests will hit the disk first (A or B).
- If the system crashes or loses power after a write is complete, the structures on disk will be left in an inconsistent state.

---

#### Crash Consistency Requirements


* Target
   * Atomically transitions the file system from one consistent state (before files are appended) to another consistent state (after inodes, bitmaps, and new data blocks have been written to disk).
* Difficulty
   * The disk commits only one write at a time, and crashes or power outages may occur between updates.

---

#### File update process example

An application updates the on-disk structure in some way: by appending a single data block to the original file.
- Appending is done by opening the file, calling `lseek()` to move the file offset to the end of the file, and then issuing a single 4KB write to the file before closing the file.

---

#### File system data structure

* Inode bitmap (inode bitmap, only 8 bits, one for each inode)
* Data bitmap (data bitmap, also 8 bits, one for each data block)
* Inodes (8 in total, numbered 0 to 7, distributed over 4 blocks)
* Data blocks (8 in total, numbered 0~7).

<!--
Schematic diagram of the file system
-->
![w:1100](figs/crash-ex.jpg)



---
<style scoped>
{
  font-size: 30px
}
</style>
#### Disk operations in file updates

An application updates the on-disk structure in some way: appending a single data block to the original file
- Must perform **3 separate writes** to disk
   - inode (I[v2]), bitmap (B[v2]) and data block (Db)
- When the write() system call is issued, these write operations usually do not happen immediately.
   - Dirty inodes, bitmaps and new data first exist in **memory** (page cache page cache, or buffer cache buffer cache) for a period of time.
- When the filesystem finally decides to write them to disk (say 5s or 30s), the filesystem will issue the necessary **write requests** to the disk.

<!--![w:900](figs/crash-ex.png)-->



---

**Outline**

1. Crash consistency problem
- crash consistency
### Crash Scenario
2. The file system checker fsck
3. Journaling file system

---

#### Crashes in file operations
Crashes may occur during file operations, interfering with these updates to disk.
* If the crash occurs after one or two of the write operations complete, but not all 3, the filesystem may be in an **interesting** (inconsistent) state.
<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)

---

#### Crash Scenario 1

Only write data blocks (Db) to disk
* **The data is on disk**, but there is no inode pointing to it, nor a bitmap indicating that the block is allocated
* **As if the write never happened**

<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)

---

#### Crash Scenario 2

Only the **updated inode** (I[v2]) was written to disk
* Inode points to disk block 5, where Db is about to be written, but Db has not yet been written
* **Read junk data from disk** (old contents of disk block 5).

<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)



---

#### Crash Scenario 3

Only the **updated bitmap** (B[v2]) is written to disk
* The bitmap indicates that block 5 is allocated, but there is no inode pointing to it
* This write will cause a **space leak** (space leak), the file system will never use block 5

<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)


---

#### Crash Scenario 4

**Inode(I[v2]) and bitmap(B[v2]) were written to disk** but no data(Db) was written
* The inode has a pointer to block 5, and the bitmap indicates that 5 is in use, so from the metadata point of view of the filesystem, everything **looks normal**
* But **disk block 5 is garbage again**.

<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)

---

#### Crash Scenario Five

**Inode (I[v2]) and data block (Db)** were written, but bitmap (B[v2]) was not written
* The inode points to the correct data on disk
* There is an **inconsistency** between the inode and the old version of the bitmap (B1)

<!-- ![w:900](figs/crash-ex.png) -->
![w:1100](figs/crash-ex-normal.jpg)

---

#### Crash Scene Six

**Bitmap (B[v2]) and data block (Db)** were written, but inode (I[v2]) was not written
* Again **inconsistency** between inode and data bitmap
* Don't know which file it belongs to since there is no inode pointing to the block

![w:1100](figs/crash-ex.png)

---

**Outline**

1. Crash consistency problem
### 2. File system checker fsck
3. Journaling file system

---

#### Crash Solution

* The file system checker fsck
* File system based on write ahead log

---

#### File system checker fsck

Early filesystems took a naive approach to crash consistency.
* Let inconsistencies happen, then fix them later (on reboot)
* Goal: Ensure file system **metadata is internally consistent**.


---
<style scoped>
{
  font-size: 30px
}
</style>
#### Superblock Check

Check whether the superblock is reasonable, mainly for sanity checks
* Make sure the filesystem size is larger than the number of allocated blocks
* Found **unreasonable (conflicting)** contents of superblock, system (or administrator) can decide to use **alternate copy** of superblock

Note: A file system with high reliability will have multiple disk sectors where super block backups are placed.

![bg right:45% 100%](figs/backup-superblock.png)

---

#### Consistency check between bitmap and inode

Scans inodes, indirect blocks, double indirect blocks, etc. for blocks currently allocated in the filesystem, generating the correct version of the allocation bitmap
* If there is any inconsistency between the bitmap and the inode, resolve it by trusting the information inside the inode
* Perform the same type of check on all inodes, making sure that all inodes that look like they are in use are marked in the inode bitmap


---

#### Inode status check

Check each inode for corruption or other problems
* Each allocated inode has a valid type field (i.e. regular file, directory, symlink, etc.)
* If there is a problem with the inode field, which cannot be easily repaired, the inode is considered suspicious and cleared by fsck, and the inode bitmap is updated accordingly.

---

#### Link Count Check

The inode link count represents the number of distinct directories that contain references (i.e. links) to this particular file.
* Scans the entire directory tree starting at the root and builds its own link count for each file and directory in the file system
* If the newly computed count doesn't match the count found in the inode, usually to fix the count in the inode
* If an allocated inode is found but no directories refer to it, it will be moved to the lost+found directory.

---

#### Duplicate Pointer Check

The case where two different inodes refer to the same block
* If an inode is clearly wrong, it may be cleared or the block it points to copied, giving each inode its own file data.
* Inodes have many error possibilities, such as inconsistent metadata within their inodes
   * The inode has a record of the length of the file, but the size of the data block it actually points to is smaller than its file length.

---

#### Bad block check

When scanning the list of all pointers, check for bad block pointers. A pointer is considered "bad" if it clearly points to something outside its valid range.
* Address points to a block larger than the partition size
* Delete (clear) this pointer from the inode or indirect block

---

#### Directory Check

Fsck has no knowledge of the contents of user files, but directories contain information in a specific format created by the filesystem itself. Perform additional sanity checks on the contents of each directory.
   - Make sure "." and ".." are preceding entries, each inode referenced in a directory entry is allocated
   - Make sure no directory is referenced more than once in the entire hierarchy.

---

#### Inadequacy of the file system checker fsck

- For very large disk volumes, it may take minutes or hours to scan the entire disk to find all allocated blocks and to read the entire directory tree.
- Possible data loss!

---

**Outline**

1. Crash consistency problem
2. The file system checker fsck
### 3. Journaling file system
- Log
- Data journaling
- Performance optimization of the journaling file system

---

#### Logging (or write-ahead logging)

Write-ahead logging
- Ideas borrowed from the world of database management systems
- In file systems, write-ahead logging is often referred to as journaling for historical reasons
- The first filesystem to implement it was [Cedar](https://www.microsoft.com/en-us/research/publication/the-cedar-file-system/)
- Many modern filesystems use this idea, including Linux ext3 and ext4, reiserfs, IBM's JFS, SGI's XFS, and Windows NTFS.

---

#### The idea of pre-writing the log

* When updating disk, before overwriting the structure, first write a small note (elsewhere on disk, in a well-known location) describing what you are going to do
* Writing this annotation is the "pre-write" part, writing it into a structure and organizing it into a "log"

---

#### Crash recovery with write-ahead logging

- By writing annotations to disk, it is guaranteed that if a crash occurs during an update (overwrite) of a structure being updated, you will be able to go back and review the annotations you made, and try again
- Know exactly what to fix (and how to fix it) after a crash without having to scan the entire disk
- Journaling greatly reduces the amount of work required during recovery by adding some work during updates

---

**Outline**

1. Crash consistency problem
2. The file system checker fsck
3. Journaling file system
- Log
### Data journaling
- Performance optimization of the journaling file system

---

#### Data journaling

![w:900](figs/ext3-journal-struct.png)
- TxB: transaction start
- TxE: transaction end
- Logical logging: the middle 3 pieces of data
---


#### Data journaling

![w:900](figs/ext3-journal-struct.png)
- Data log written to disk
- Update disk, overwrite related structure (checkpoint)
   - I[V2] B[v2] Db

---

#### Crash during log write

Disk internals can (1) write to TxB, I[v2], B[v2] and TxE before writing to Db.
* If the disk is powered off between (1) and (2), then the disk becomes:

![w:1000](figs/ext3-journal-struct-err.png)

---

#### Two-step transactional write to the data log

To avoid this problem, the file system issues transactional writes in two steps.
- Write all blocks except TxE blocks to the journal while issuing these writes
- When these writes are complete, the log will look like this (assuming another file append workload):
![w:1000](figs/ext3-journal-write.png)

---

#### Two-step transactional write to the data log

When these writes are complete, the filesystem issues writes of the TxE block, leaving the journal in a final safe state:
![w:900](figs/ext3-journal-commit.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Data log update process

The current protocol for updating the filesystem is as follows, with names for each of the 3 phases.
1. **Log write**
    - Write the contents of the transaction (including TxB, metadata and data) to the log, wait for these writes to complete.
2. **Log submission**
    - Write the transaction commit block (including TxE) to the log, wait for the write to complete, and the transaction is considered committed.
3. **Add Checkpoint**
    - Write updates (metadata and data) to their final disk locations.

---

#### Crash Recovery for Data Logs

A crash can occur at any time during this update sequence.

- If the crash occurred before the transaction was safely written to the log
- If the crash occurred after the transaction was committed to the log but before the checkpoint completed

Too much to write, slow down!

---

**Outline**

1. Crash consistency problem
2. The file system checker fsck
3. Journaling file system
- Log
- Data journaling
### Performance optimization for journaling filesystems

---

#### Journal superblock

- Batch log update
- Make log limited: circular log

![w:900](figs/ext3-journal-batch.png)
![w:900](figs/ext3-journal-superblock.png)

---

#### The update process of the log super block
- Journal write
- Journal commit
- Checkpoint
- Free: After a period of time, by updating the journal, the superblock marks the transaction as free


---

#### Metadata Journaling

When should data block Db be written to disk?
   - The order in which data is written is important for metadata-only logging
   - Is it a problem if Db is written to disk after the transaction (contains I[v2] and B[v2]) is complete?

---

#### Metadata log update process
- Data write
- Journal metadata write
- Journal commit
- Checkpoint metadata
- Free

By forcing data to be written first, the file system guarantees that pointers never point to garbage data.


---

#### Data Journaling timeline v.s. Metadata Journaling timeline
![bg 90%](figs/ext3-journal-data-timeline.png)


![bg 90%](figs/ext3-journal-metadata-timeline.png)

---
<style scoped>
{
  font-size: 32px
}
</style>
### Summary

1. Crash consistency problem
- Crash consistency
- Crash scene
2. The file system checker fsck
3. Journaling file system
- Log
- Data journaling
- Performance optimization of the journaling file system