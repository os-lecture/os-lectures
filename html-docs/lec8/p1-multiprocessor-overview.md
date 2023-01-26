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

# Lecture 8 Multiprocessor Scheduling

## Section 1 Symmetric multiprocessing and multi-core architecture


<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Multiprocessing machines
2. Cache consistency (Cache Coherence)

---

#### Single Core Processor
![w:800](figs/single-core.png)


---
#### Hyperthread (Hyperthread, Simultaneous multithreading) processor
![w:500](figs/hyperthread.png)


---
#### Multi-core processor
![w:1150](figs/multi-core.png)

---
#### Many-core processors
![w:1150](figs/many-core.png)

---

**Outline**

1. Multiprocessing machines
### 2. Cache consistency (Cache Coherence)

---

#### Symmetrical multiprocessor (SMP) and non-uniform memory access system (NUMA)
![w:900](figs/smp-numa.png)

---
#### Cache consistency (Cache Coherence)
![w:800](figs/cache-coherence.png)


---

#### Cache consistency problem
![w:900](figs/cache-coherence-problem.png)