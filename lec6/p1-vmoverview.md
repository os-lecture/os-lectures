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

# Lecture 6 Virtual storage management
## The first section virtual storage concept

<br>
<br>
<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Requirements for virtual storage technology
2. Overlay technology
3. Switch technology
4. Basic concepts of virtual storage
5. Page fault exception

---

#### Requirements background of virtual storage technology

The growth rate of the program size is much faster than the growth rate of the memory capacity
![w:700](figs/game-mem.png)
Ideal Memory: **Non-Volatile** memory with **larger **capacity**, **faster** speed, **cheaper **price**


---
<style scoped>
{
  font-size: 28px
}
</style>

#### Basic idea of virtual storage
**Challenge**: The computer system often encounters **not enough memory**
**Thinking**: Insufficient memory, use external storage to make up
- Function coverage (overlay)
   - The application is **manually** swapped in and out of memory in units of **function/module**
- program exchange (swapping)
   - The operating system automatically swaps in and out of memory in units of **programs**
- virtual storage
   - The operating system automatically swaps in and out of memory in units of **page**

Virtual storage = memory + external storage

---
#### Address space
<!--The computer system often has insufficient memory space
- Module overlay (overlay)
   - The application **manually** saves the required commands and data in memory
- Task exchange (swapping)
   - The operating system **automatically** saves temporarily unexecutable programs to external memory
- virtual storage
   - Automatically load more and larger programs in units of **pages in limited memory
--->
The address space is the **abstraction** of virtual storage by the operating system.
![w:800](figs/os-abstract-address-space.png)



---

**Outline**

1. Requirements for virtual storage technology
### 2. Overlay technology
3. Switch technology
4. Basic concepts of virtual storage
5. Page fault exception

---

#### Coverage Techniques
- Target
   - Programmer **manually** controlled to run **bigger programs** in **smaller available memory**
- The basic idea
   - **Functions or modules** executed in **different time periods** share a limited space


![bg right:48% 75%](figs/overlay-simple.png)

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Fundamentals of coverage
   Overlay refers to dividing a program into a series of program segments with relatively independent functions, so that program segments that are not required to be loaded into memory at the same time during execution form a group (called an overlay segment), and **shared **Same area of main memory.
   - **Necessary** part of (commonly used) code and data resident in memory
   - **Optional** parts (not commonly used) are placed in other program modules, and only loaded into memory when **needed**
   - Modules with **no calling relationship** can overwrite each other and share the same memory area

---

#### Coverage Technique Example

![w:900](figs/overlay.png)
 


---

#### Insufficient overlay technology

- Added **Program Difficulty**
   - Programmers are required to divide functional modules and determine the coverage relationship between modules
   - Increased programming complexity;
- Increased **execution time**
   - Load overlay module from external memory
   - time for space

  Overlay system unit of Turbo Pascal supports programmer-controlled overlay technology


---

**Outline**

1. Requirements for virtual storage technology
2. Overlay technology
### 3. Exchange Technology
4. Basic concepts of virtual storage
5. Page fault exception

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Exchange Technology
- The basic idea
   - The operating system automatically swaps in and out of memory in units of **programs**
- Method
   - **Swap out**: Save the entire address space content of an executing program to external memory
   - **Swap in (swap in)**: read the address space content of an execution program in the external memory into the memory

![bg right:30% 100%](figs/swapping.png)

---

#### Problems faced by switching technology

- Swap **timing**: When does the swap need to happen?
   - Only swap out when there is not enough memory space or there is a possibility of not enough
- Swap area **size**: store copies of all memory images of all user processes
- **Relocation** when the program is swapped in: Should it be placed in the same place when swapped out and then swapped in?
   - Not necessarily in place, some mechanism is needed to ensure correct addressing & execution of the program


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Comparison of Overwrite and Swap
- program override
   - Occurs between **modules/functions** that are not on a control flow for a certain period of time
   - in units of modules/functions
   - The programmer must give the **logical coverage structure** between modules/functions
- exchange
   - Occurs between running **programs**
   - In units of running programs
   - No need for logical overlay structures between modules

Program to run: ``task`` or ``process``
 
---

**Outline**

1. Requirements for virtual storage technology
2. Overlay technology
3. Switch technology
### 4. The basic concept of virtual storage
5. Page fault exception

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Definition of virtual storage
- Definition
   - Virtual Storage = **Memory** + **External Storage**
- Ideas
   - The operating system **temporarily stores** part of the memory that is not frequently used** to the external storage, and loads the data to be accessed by the processor from the external storage ** into the ** memory
- **Prerequisites**
   - Program has **locality**

![bg right:40% 95%](figs/virtual-memory.png)

---
<style scoped>
{
  font-size: 28px
}
</style>

#### Principle of Locality

Locality (locality): A **short period** during the execution of the program, the executed **instruction** address and the **operand** address of the instruction are respectively limited to a certain area
- **Time** Locality: One execution and next execution of **one instruction**, one access and next access of **one data** are all concentrated in a short period of time
- **Space** locality: the current instruction and several instructions of **adjacent time**, the currently accessed data and several data accessed by adjacent time are concentrated in a small area
- **Branch** locality: two executions of a **jump instruction** are likely to jump to the same memory location

Significance of locality: If most programs run with local characteristics, virtual storage technology can be realized and satisfactory results can be achieved
 


---
<style scoped>
{
  font-size: 28px
}
</style>

#### Ideas and rules of virtual storage
- Idea: Temporarily store some memory blocks that are not frequently used in external storage
- Rule:
   - When **loading** the program: only load the **partial pages or segments required by the **current instruction execution into the memory
   - When the instruction or data **needed in the instruction execution is not in the memory** (called a page or segment): the processor notifies the operating system to transfer the corresponding page or segment into the memory
   - The operating system saves pages or segments that are **temporarily unused** in memory to external memory
- Method to realize:
   - virtual page storage
   - Virtual segment storage


---
#### Basic characteristics of virtual storage
- Discontinuity
   - Physical memory allocation is non-contiguous
   - The virtual address space uses non-contiguous
- Large user space
   - The virtual memory provided to the user can be larger than the actual physical memory
- Partial exchange
   - Virtual storage only transfers in and out of part of the virtual address space
 

---
#### The underlying support of virtual storage
- Hardware (MMU/TLB/PageTable)
   - Hardware **address translation** mechanism in page or segment storage, hardware **exception**
- Software (OS)
   - Create **page table or segment table** in memory
   - Manage **swap-in and swap-out** of pages or segments between memory and external memory
 

---
<style scoped>
{
  font-size: 28px
}
</style>

#### Virtual paging storage management
On the basis of paging storage management, add demand paging and page replacement

- **The basic idea**
   - When the user program is to be loaded into the memory, only **load some** pages, and start the program to run
   - When the user program finds that the required code or data is not in the memory during operation, it will send a **page fault exception** request to the system
   - When the operating system handles a page fault exception, it transfers the corresponding page in the external memory to the memory, so that the user program can continue to run
   - When the memory is almost used up, the operating system transfers some pages from the memory** to the external memory


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Virtual paging storage management
On the basis of paging storage management, add demand paging and page replacement
- Request paging: also known as on-demand paging, when the processor needs to access certain data, the data is transferred from the external storage to the memory
- Page replacement: swap out infrequently used pages and swap in pages to be used
- Page fault exception handling: **Software and hardware coordination support**
 
![bg right:40% 100%](figs/vm-work.png)


---

**Outline**

1. Requirements for virtual storage technology
2. Overlay technology
3. Switch technology
4. Basic concepts of virtual storage
### 5. Page fault exception

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Processing flow of page fault exception
1. The CPU reads the memory unit, matches the physical address in the TLB according to its virtual address, misses, **reads the page table**;
1. Since the existence bit of the page table entry is 0, the CPU generates a **page fault exception**;
1. The OS **finds** the page content of the corresponding application stored in the external memory;

![bg left:40% 100%](figs/page-fault-handler.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Processing flow of page fault exception

4-1. If there is an idle physical page frame, swap the page content** in the external memory into an idle physical page frame**;
4-2. If there is no free physical page frame, release/swap out a physical page frame to the external memory through the replacement algorithm, and then swap the page content in the external memory into a free physical page frame;

![bg left:40% 100%](figs/page-fault-handler.png)

---

#### Processing flow of page fault exception

5. **Modify the page table entry**, establish the mapping from the virtual page to the physical page frame, and the existence position is 1;
6. The OS returns to the application program and asks the processor to **re-execute** the read memory unit instruction that caused the page fault exception.

![bg left:40% 100%](figs/page-fault-handler.png)
* Where are unmapped pages kept? How to find this page? *

---
#### Where to store unmapped pages?
   - Swap space (disk/file form)
     - Store unmapped pages in a special format
   - a file (code or data) on disk
   ![bg right:50% 100%](figs/page-fault-handler.png)
---
<style scoped>
{
  font-size: 30px
}
</style>

#### External memory swap space for virtual storage

Where to save the address of the page placed in external memory?

- Swap space
     - Disk partition: generally sector address
     - Save the page address of the external memory in the page table entry whose existence bit is 0
     

![bg right:50% 100%](figs/page-fault-handler.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### External storage disk file for virtual storage

Where to save the address of the page placed in external memory?

- a file (code or data) on disk
   - The logical segment representation in the address space has a corresponding file location
     - eg: `MemorySet::MapArea`
   - Code snippets: executable binaries
   - Dynamically loaded shared library program segment: dynamically called library file

![bg right:40% 100%](figs/page-fault-handler.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Virtual Storage Performance

Effective memory access time (EAT, Effective memory Access Time)
- EAT = memory access time $*$ (1-p) + page fault exception handling time
- Page fault processing time = disk access time*p(1+q)
   - p: page fault rate;
   - q: writeback probability
* Example
   - Memory access time: 10 ns; Disk access time: 5 ms
   - EAT = 10(1–p) + 5,000,000p(1+q)

---

### Summary

1. Requirements for virtual storage technology
2. Overlay technology
3. Switch technology
4. Basic concepts of virtual storage
5. Page fault exception