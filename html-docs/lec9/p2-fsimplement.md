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

## Section 2 Design and Implementation of File System


<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1 Overview
2. The basic data structure of the file system
3. File caching
4. File distribution
5. Example of file access process

---
#### The location of the file system in the kernel
![w:850](figs/ucorearch.png)

---

#### Hierarchical structure of the file system

![w:950](figs/fsarch.png)

---
#### The hierarchical structure of the file system in the computer system

![w:700](figs/fsarchdetail.png)

---

#### User view and kernel view of the file system

![w:450](figs/fslayer.png)

![bg right:55% 95%](figs/fslayout.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### Virtual file system

(Virtual File System, VFS)

- Defines a set of data structures and standard interfaces supported by all file systems.
- Disk file system: directly store data in the disk, such as Ext 2/3/4, XFS.
- In-memory filesystems: in-memory auxiliary data structures - such as directory entries.
![bg right 50% 80%](figs/fsarchi.png)


---
#### Virtual file system function
- Purpose: Abstraction of all the different filesystems
- Features
   - Provides the same file and filesystem interface
   - Manages all file and file system associated data structures
   - Efficient query routines, traversing the file system
   - Interaction with specific filesystem modules


---
#### The virtual file system unifies the access interfaces of different file systems
![w:700](figs/vfs-app.png)


---

**Outline**

1 Overview
### 2. The basic data structure of the file system
3. File caching
4. File distribution
5. Example of file access process

---

#### Storage view of the file system
- File volume control block (`superblock`)
- File control blocks ( `inode`/`vnode`)
- directory entry (`dir_entry`)
- Data block (`data block`)
![bg right:45% 100%](figs/fsdisk.png)

---

#### Organizational view of the file system

![bg 90%](figs/fsorg.png)
![bg 90%](figs/fsall.png)


---

#### The basic data structure of the file system
![w:700](figs/fsoverall.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### Volume control block (`superblock`)

One file volume control block per file system
- File system details
- Block size, number of free blocks, etc.
- Total number of blocks and inodes, unused and used
- Filesystem mount time, last write time, last check disk (fsck) time
![bg right:45% 100%](figs/efs-superblock.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### File control block inode
Each file has a file control block inode (`inode`/`vnode`)
   - Size, data block location (pointing to one or more datablocks)
   - Access mode (read/write/excute)
   - Owner and group (owner/group)
   - Time information: time of establishment or state change, last read time/modified time
   - **Filename is in the datablock of the directory**
<!--![bg right 10% 50%](figs/efs-inode.png)-->

![bg right:46% 90%](figs/efs-inode.png)



---
#### Bitmap block
Bitmap block ( `bitmap inode/dnode`)
- Inode used or unused flag
- Dnode use or not use flag

---
#### Data block dnode( `data node`)
- Data blocks for directories and files
     - Put directory and file content
     - Determine the fixed size of the data block when formatting
     - Each block has a number to facilitate inode records
     - Inode is generally 128B
     - The data block is generally 4KB

---
#### Data blocks for directories
![w:850](figs/fsdir.png)



---
<style scoped>
{
  font-size: 30px
}
</style>
#### Directory Entry (`dir_entry`)
- Directory entries are generally cached in memory
   - one per directory entry (directory and file)
   - Encode directory entry data structure and tree layout into a tree data structure
   - points to file control blocks, parent directories, subdirectories, etc.
![bg right 100%](figs/efs-direntry.png)

![bg right 100%](figs/fslayout.png)

---

**Outline**

1 Overview
2. The basic data structure of the file system
### 3. File cache
4. File distribution
5. Example of file access process

---

#### Various disk cache locations
![w:1200](figs/diskcache.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Data block cache
- Data blocks **on-demand read** into memory
   - Provide read() operation
   - Read ahead: read the following data blocks in advance
- Data blocks are **cached** after use
   - Assuming the data will be used again
   - Write operations may be cached and delayed
 
  Page cache: Unified cache of data blocks and memory pages

![bg right:45% 90%](figs/datacache.png)

---
#### Virtual page storage -- page cache

In the virtual address space, virtual pages can be mapped to local external memory files
  
![w:700](figs/pagecache.png)

<!--
---
#### Design and implementation of the file system -- cache
Virtual page storage -- page cache
- Virtual pages in the virtual address space can be mapped to local external memory files
- Page cache for file data blocks
   - File data blocks are mapped into pages in virtual memory
   - File read/write operations are converted to memory accesses
   - May result in page faults and/or dirty pages
   - Problem: The page replacement algorithm needs to coordinate the number of pages between the virtual storage and the page cache
-->


---
#### Virtual page storage -- page cache

In the virtual address space, virtual pages can be mapped to local external memory files
- Page cache for file data blocks
   - File data blocks are mapped into pages in virtual memory
   - File read/write operations are converted to memory accesses
   - May result in page faults and/or dirty pages
- Problem: The page replacement algorithm needs to coordinate the number of pages between the virtual storage and the page cache


---
#### File descriptor
- Each opened file has a file descriptor
- As an index, it points to the corresponding file status information

![w:750](figs/fd-openfiletable.png)



---
#### Open file table
- One process open file table per process
- A system open file table

![w:750](figs/fd-openfiletable.png)


---
#### File Lock
Some file systems provide file locks for coordinating file access by multiple processes
- Mandatory – Deny access based on lock hold and access requirements
- Advisory - Processes can look up the state of the lock to decide what to do


---

**Outline**

1 Overview
2. The basic data structure of the file system
3. File caching
### 4. File allocation
5. Example of file access process

---

#### File size
- Most files are small
   - need to support small files
   - The data block space cannot be too large
- Some files are very large
   - Can support large files
   - Efficient reading and writing
![bg right:50% 90%](figs/fd-openfiletable.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### File Allocation

Allocate file data blocks
- Allocation
    - continuous allocation
    - chain allocation
    - index assignment
- Evaluation indicators
   - Storage efficiency: external fragmentation, etc.
   - Read and write performance: access speed

![bg right:50% 90%](figs/fd-openfiletable.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Continuous Allocation
The file header specifies the starting block and length

![w:900](figs/continuealloc.png)

- Assignment strategy: first match, best match, ...
- advantage: 
   - Efficient sequential and random read access
- Shortcoming
   - Frequent allocation will bring fragmentation; adding file content is expensive



---
#### Chain allocation
Data blocks are stored in a linked list

![w:800](figs/linkalloc.png)

- Pros: easy to create, grow, shrink; almost no fragmentation
- Shortcoming:
    - Low random access efficiency; poor reliability;
    - Breaking a chain, the following data blocks are lost


---
#### Chain allocation

   - Explicit link
   - Implicit connection
![bg right:35% 80%](figs/fs-explicit.png)

![bg right:70% 100%](figs/fs-implicit.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### Index Assignment

- The file header contains the index data block pointer
- The index in the index data block is a pointer to the file data block
![w:800](figs/indexalloc.png)
- Advantage
   - Easy to create, grow, shrink; almost no fragmentation; supports direct access
- Shortcoming
   - When the file is small, the overhead of storing the index is relatively large

How to handle large files?


---
#### Index Assignment

- Chained index blocks (IB+IB+…)
![w:800](figs/linkindex.png)
- Multi-level index blocks (IB*IB *…)
![w:800](figs/multiindex.png)


---
#### Index Assignment

![w:1000](figs/fsindex.png)


---
#### Multi-level index allocation

![w:800](figs/ufsinode.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Multi-level index allocation

- The file header contains 13 pointers
   - 10 pointers to data blocks
   - The 11th pointer points to the index block
   - The 12th pointer points to the secondary index block
   - The 13th pointer points to the third-level index block

Large files require a lot of queries when accessing data blocks


![bg right:43% 100%](figs/ufsinode.png)


---
#### Comparison of file allocation methods
![w:1150](figs/filespace.jpg)



---
<style scoped>
{
  font-size: 32px
}
</style>
#### Free space management
Tracking unallocated data blocks in a file volume: data structure?
- Bitmap: Use a bitmap to represent the list of free data blocks
   - 11111111001110101011101111...
   - $D_i = 0$ indicates that the data block $i$ is free, otherwise, it means allocated
   - 160GB disk --> 40M data block --> 5MB bitmap
   - assuming free space is evenly distributed across the disk,
       - scan n/r before finding "0"
         - n = total number of data blocks on disk; r = number of free blocks

---
#### Free space management
- linked list
![w:800](figs/link.png)
- index
![w:900](figs/linkindex4free.png)
 


---

**Outline**

1 Overview
2. The basic data structure of the file system
3. File caching
4. File distribution
### 5. Example of file access process

---

#### File system organization example
![w:850](figs/fsall.png)

---
#### File read operation process
![w:650](figs/fsread.jpg)

---
#### File write operation process
![w:800](figs/fswrite.jpg)



---
#### File system partition
- Most disks are divided into one or more partitions, each with a separate file system.
![w:600](figs/fs-oall.png)

![bg right:40% 80%](figs/fs-block.jpg)

---

### Summary

1 Overview
2. The basic data structure of the file system
3. File caching
4. File distribution
5. Example of file access process