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

### Section 3 Historical evolution of the operating system

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---

## Single User System

Single User Systems (1945-1955)

- **Manual** wire/tape transfer for program entry
- The machine cost is much greater than the labor **cost**
- **OS = loader + libraries**
- Problem: **Low utilization** of expensive components

![bg right:40% width:520](./figs/history-single-user-system.png)

---

## Batch processing system

Batch systems (1955-1965)

- **TAPE/DISK TRANSFER** for program input
- Machine cost is greater than labor cost
- Operating System = Loader + **Program Controller + Output Processor**
- Problem: Increased utilization compared to before

![bg right:35% 100%](./figs/history-batch-processing.png)

---

## Batch processing system

**Batch Processing** Systems (1955-1965)

- Tape/disk transfer for program input
- Machine cost is greater than labor cost
- Operating system = loader + program controller + output processor
- Problem: Increased utilization compared to before

![bg right:35% 100%](./figs/history-batch-process-graph.png)

---

## Multiprogramming system

**Multi-channel** programming system (1955-1980)

- multiple programs reside in **memory**
- Multiple programs take turns using **CPU**
- OS = loader + **program scheduling + memory management** + output management
- Evolution: Increased utilization compared to previous

![bg right:35% 100%](./figs/history-multiprogramming.png)

---

## Time Sharing System

Time Sharing System (1970-present)
- Multiple programs reside in memory
- Multiple programs time-sharing use of the CPU
- Operating system = loader + program scheduling + memory management + **interrupt handling** +...
- Evolution: Compared with the previous utilization rate, the interaction delay with the outside world is shortened

![bg right:35% 100%](./figs/history-timesharing.png)

---
## Multics OS

![](./figs/history-multics.png)

---
## Multics OS

![](./figs/multics-intro.png)

---
## Open UNIX

![bg 50%](./figs/unix-family.png)


---
## Linux family

![bg 55%](./figs/linux-family.png)

---
## Personal computer

Personal computer (1981- )
- Single user
- **Computer costs drop** so that CPU utilization is no longer a top concern
- Focus on **User Interface and Multimedia Features**
- Operating system = loader + program scheduling + memory management + interrupt handling +...
- Evolution: **to the public**, old services and features don't exist, more and more security issues

![bg right:20% 100%](./figs/history-pc.png)

---
## MacOS Family

![bg 55%](./figs/macos-family.png)

---
## MacOS Family

![bg 55%](./figs/macos-family-history.png)

---
## Windows family

![bg 70%](./figs/windows-family.png)

---
## Distributed Systems

**Distributed** System (1990- )
- Distributed multi-user
- Distributed system utilization is a concern
- The focus is on network/storage/compute efficiency
- OS = distributed (loader + program/OS scheduling + memory management)
- Evolution: to the public, towards **Internet**, new challenges (unreliable/uncertain)

![bg right:30% 100%](./figs/history-ds.png)

---
##Android OS
- Cross-platform: supports Java applications
- Runtime (runtime): Android virtual machine
- Application Framework: Simplifies application development


![bg right 80%](./figs/android-system-architecture.png)

---
## AIoT operating system

AIoT system (2000- )
- Distributed **multi-device**
- Distributed system utilization/availability is a concern
- The focus is on network/storage/compute efficiency
- OS = distributed (program/OS scheduling + memory management + security/updates)
- Evolution: towards device, towards network, new challenges (unreliable/big data)


![bg right:28% 100%](./figs/history-aiot.png)

---
## Fuchsia OS

![bg 65%](./figs/fuchsia-os-intro.png)