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
<!-- page_number: true -->
<!-- _class: lead -->

## Lecture 1 Operating System Overview

### Section 4 Operating System Structure

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---
## Simple structure
MS-DOS: Application and OS mixed together (1981–1994)
- **Not split into modules**
- Written primarily in assembly
- No security

![bg right 100%](./figs/msdos.png)


---
## Monomer Hierarchy
Divide the Monolithic OS into **multi-layers** (levels)
- Each layer builds on top of the lower
- The bottom layer (layer 0), is the hardware driver
- The highest layer (layer N) is the user interface
- Each layer uses only the functions and services of the lower layer

![bg right:38% 100%](./figs/multi-level-os-arch.png)


---
## Micro Kernel Structure (Micro Kernel)
- **Move kernel functions to user space as much as possible**
- Communication between user modules uses message passing
- Benefits: flexible/safe...
- Cons: performance
- LPC: Local Procedure Call (Local Procedure Call)
- HAL: Hardware Abstraction Layer (Hardware Abstraction Layer)

![bg right:23% 100%](./figs/microkernel-arch.png)

---
## Exokernel
- Let the kernel allocate physical resources to multiple applications, and let each program decide what to do with those resources
- Programs can link to the operating system library (libOS) which implements the operating system abstraction
- **Separation of protection and control**
- Distributed Shared Memory(DSM)

![bg right:35% 100%](./figs/exokernel-arch.png)


---
## Virtual machine structure
The virtual machine manager converts a single machine interface into many virtual machines, each virtual machine is an effective copy of the original computer system and can execute all processor instructions

![bg right:45% 100%](./figs/vmm-arch.png)

---
## Virtual machine structure


![bg 65%](./figs/vmm-arch-view2.png)

---
## Relationship between application running and OS Abstraction + Architecture

![bg 75%](./figs/os-env.png)