---
marp : true
theme : default
paginate : true
_paginate : false
header : ''
footer : ''
backgroundColor : white
---

<!-- theme: gaia -->
<!-- _class: lead -->

# Lecture 6 Virtual Storage Management
## Section 2 Partial Page Replacement Algorithm



Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---
<style scoped>
{
  font-size: 28px
}
</style>

#### The function and design goal of the page replacement algorithm

- Function
  - When a page fault occurs and a new page needs to be transferred and the memory is full, the replacement algorithm **selects the replaced physical page**

- Design goals
  - Minimize page **number of missing pages** , **swap-in/swap-out times**
  - Bring out pages that will no longer be visited in the future or **not visited in the short term**

![ bg right:40% 100% ]( figs/page-fault-handler.png )

---
#### Timing of page replacement

- **Free memory** upper limit and lower limit
- Reach the lower limit and start reclaiming memory
- Reach the upper limit, suspend memory recovery

![ bg right:50% 100% ]( figs/reclaim-page.png )

---

#### Frame locking/resident memory

**Logical pages that must be resident in memory**
- Critical parts of the operating system
- Code and data that require responsiveness
- The lock bit in the page table (lock bit)

![ bg right:53% 100% ]( figs/page-fault-handler.png )

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Evaluation Method of Page Replacement Algorithm

- Evaluation method
  - Record the track of the process **accessing the memory page** , simulate the replacement behavior, and record the number of page faults** ;
  - Fewer **fewer** page faults, better ** better** performance
- Example : Virtual address access is represented by (page number, displacement)
```
(3,0), (1,9), (4,1), (2,1), (5,3), (2,0), (1,9), (2,4), (3 ,1), (4,8)
```
- Corresponding page track
```
3, 1, 4, 2, 5, 2, 1, 2, 3, 4 in numbers
c, a, d, b, e, b, a, b, c, d in characters
```

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Classification of Page Replacement Algorithms

- **Partial** page replacement algorithm
  - The selection range of the replacement page is limited to the physical page occupied by **current process**
  - Optimal algorithm, first-in-first-out algorithm, least recently used algorithm
  - Clock algorithm, least commonly used algorithm

- **Global** page replacement algorithm
  - The selection range of the replacement page is **all** physical pages that can be swapped out
  - Working set algorithm, page fault rate algorithm

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
### 2. Optimal Page Replacement Algorithm (OPT, optimal)
1. First in first out page replacement algorithm (FIFO)
2. The least recently used page replacement algorithm (LRU, Least Recently Used)
3. Clock page replacement algorithm (Clock)
4. Improved clock page replacement algorithm
5. Least Frequently Used Algorithm (LFU, Least Frequently Used)
6. The Belady phenomenon

---

#### How the Optimal Page Replacement Algorithm Works
- Basic idea
  - Replace pages that will not be visited in the **maximum time** in the future
- Algorithm implementation
  - When there is a page fault, calculate the **next access time** for each logical page in memory
  - Select the page you will not visit for the longest time in the future
---
#### Optimal Page Replacement Algorithm Features
- Minimal missing pages, **ideal situation**
- **Unrealizable** in actual system
- **Unpredictable** Waiting time for each page before next visit
  - Run a program on the emulator and record every page visit
  - Use the optimal algorithm in the second run
---

#### Optimal Page Replacement Algorithm Example

![ w:1000 ]( figs/opt-1.png )

---

#### Optimal Page Replacement Algorithm Example

![ w:1000 ]( figs/opt-2.png )

---

#### Optimal Page Replacement Algorithm Example

![ w:1000 ]( figs/opt-3.png )

---

#### Optimal Page Replacement Algorithm Example

![ w:1000 ]( figs/opt-4.png )

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
### 3. First-in-first-out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---
<style scoped>
{
  font-size: 32px
}
</style>

#### How the FIFO algorithm works
- Basic idea
  - Select the page with the longest resident time in memory for replacement
- Algorithm implementation
  - Maintain a linked list of logical pages that record all **located in memory**
  - The elements of the linked list are **sorted by the time of resident memory** , the chain head is the longest and the chain tail is the shortest
  - When a page is missing, **select** the first page of the chain to replace, and the new page is added to the end of the chain

---
#### First-in-first-out algorithm features

- Easy to implement
- The performance is poor, and the pages called out may be frequently accessed
- When the number of allocated physical pages increases, page faults do not necessarily decrease (Belady phenomenon)
- Rarely used alone

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-1.png )

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-2.png )

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-3.png )

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-4.png )

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-5.png )

---

#### First in first out algorithm example

![ w:900 ]( figs/fifo-6.png )

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
### 4. The least recently used algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---
<style scoped>
{
  font-size: 28px
}
</style>

#### How the least recently used algorithm works

- Basic idea
  - Select the page that has not been referenced for the longest time to replace
  - If a page has not been visited for a long time, it **maybe** will not be visited for a long time in the future

- Algorithm implementation
  - When there is a page fault, calculate the **last access** time for each logical page in memory
  - Select the page that has been used for the longest time from the last time to replace it
- Algorithm features
  - **An approximation of the optimal permutation algorithm**

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-1.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-2.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-3.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-4.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-5.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-6.png )

---

#### Example of algorithm that has not been used for the longest time recently

![ w:900 ]( figs/lru-7.png )

---
<style scoped>
{
  font-size: 30px
}
</style>

#### LRU page linked list implementation

- Page linked list
  - The system maintains a **linked list** of pages sorted by the last access time
    - **The first node of the linked list** is the page that has just been used recently
    - **The tail node of the linked list** is the page that has not been used for the longest time
  - When accessing memory, find the corresponding page and **move it to the head of the linked list**
  - When a page is missing, replace the page at the tail node of the linked list
- Features
  - Expensive

---

#### Active page stack implementation of LRU

- Activity page **stack**
  - When accessing a page, push this page number **onto the top of the stack** , and extract the same page number from the stack
  - When a page is missing, **replace the page at the bottom of the stack**
- Features
  - Expensive

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-1.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-2.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-3.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-4.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-5.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-6.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-7.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-8.png )

---

#### LRU active page stack implementation example

![ w:900 ]( figs/stack-lru-9.png )

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
### 5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---
<style scoped>
{
  font-size: 30px
}
</style>

#### How the Clock Replacement Algorithm Works

- Basic idea
  - Only make **approximate statistics** on the visits of the page
- Data structure
  - Add **Access Bit** to the page table entry to describe the access status of the page in the past period of time
  - Each page is organized into a **circular linked list**
  - The pointer points to **the first loaded page**

![ bg right:40% 100% ]( figs/clock-demo.png )

---


#### How the Clock Replacement Algorithm Works

- Algorithm implementation
  - **When accessing the page** , record the page access status in the page table entry
  - **When a page is missing** , start from the pointer to sequentially search for pages that have not been accessed for replacement

![ bg right:30% 100% ]( figs/clock-demo.png )

---
<style scoped>
{
  font-size: 27px
}
</style>

#### The specific implementation process of the clock replacement algorithm

- When a page **loads into memory** , the access bit is initialized to 0
- **When accessing a page (read/write)** , access position 1
- **When a page is missing** , check sequentially from the current position of the pointer
  - When the access bit is 0, replace the page
  - When the access bit is 1, the access position is 0, and the pointer moves to the next page until a replaceable page is found
- Algorithm features
  - The clock algorithm is a compromise between LRU and FIFO

![ bg right:30% 100% ]( figs/clock-demo.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-1.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-2.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-3.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-4.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-5.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-6.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-7.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-8.png )


---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-9.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-10.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-11.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-12.png )


---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-13.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-14.png )

---

#### Clock Replacement Algorithm Example

![ w:900 ]( figs/clock-15.png )


---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
### 6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---
<style scoped>
{
  font-size: 30px
}
</style>

#### How the Improved Clock Replacement Algorithm Works

- Basic idea
  - Reduce page fault handling overhead for **modified pages**
- Data structure
  - Add **Modification Bit** to the page to describe the write access of the page in the past period of time
- Algorithm implementation
  - When accessing a page, record the page access status in the page table entry
  - **When modifying the page** , record the page modification in the page table entry
  - When a page is missing, modify the page flag bit to **skip** the modified page
---

#### How the Improved Clock Replacement Algorithm Works

![ w:1150 ]( figs/advanced-clock-demo.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-1.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-2.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-3.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-4.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-5.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-6.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-7.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-8.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-9.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-10.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-11.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-12.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-13.png )

---

#### Improved Clock Replacement Algorithm Example

![ w:1000 ]( figs/aclock-14.png )

---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
### 7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon

---

#### How the Least Common Algorithm Works

- Basic idea
  - When a page is missing, replace the page with the **least visits**

- Algorithm implementation
  - Set a **visit count** per page
  - When a page is visited, **Visit count is incremented by 1** 
  - When a page is missing, replace **the page with the smallest count**

---

#### Least Common Algorithm Features

- Features
  - Algorithmic overhead is high
  - Pages that are frequently used at first but not used later are **difficult to replace**
    - Workaround: count shifted right periodically

- LRU pays attention to how long you have not visited, the shorter the time, the better
- LFU focuses on the number of visits, the more the better

---

#### LFU Example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-1.png )

---

#### LFU example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-2.png )

---

#### LFU example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-3.png )

---

#### LFU example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-4.png )

---

#### LFU example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-5.png )

---

#### LFU Example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-6.png )

---

#### LFU example

4 physical page frames, the initial number of visits a->8 b->5 c->6 d->2
![ w:1100 ]( figs/lfu-7.png )

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
### 8. The Belady Phenomenon

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Belady Phenomenon
- Phenomenon
  - When FIFO and other algorithms are used, there may be an abnormal phenomenon that **the number of allocated physical pages increases** and the number of ** page faults increases**
- Reason
  - The replacement feature of the FIFO algorithm is inconsistent with the dynamic feature of the process accessing the memory **contradiction**
  - The pages replaced by it are **not necessarily** the process will not visit in the near future
- Thinking
  - Which permutation algorithms do not have the Belady phenomenon?

---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-1.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-1.png )

---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-2.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-3.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-4.png )

---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-5.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 3; Number of page faults: 9
![ w:1100 ]( figs/belady-3fifo-6.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 4; Number of page faults: 10
![ w:900 ]( figs/belady-4fifo-1.png )


---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 4; Number of page faults: 10
![ w:900 ]( figs/belady-4fifo-2.png )



---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 4; Number of page faults: 10
![ w:900 ]( figs/belady-4fifo-3.png )



---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 4; Number of page faults: 10
![ w:900 ]( figs/belady-4fifo-4.png )



---

#### Belady phenomenon of FIFO algorithm

Access sequence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of physical pages: 4; Number of page faults: 10
![ w:900 ]( figs/belady-4fifo-5.png )


---

#### FIFO algorithm has Belady phenomenon

![ w:600 ]( figs/belady-3fifo-6.png )
![ w:600 ]( figs/belady-4fifo-5.png )
<!--![bg left:50% 100%](figs/belady-3fifo-7.png)
![bg right:50% 100%](figs/belady-4fifo-5.png)-->
<!--![w:900](figs/belady-4fifo-5.png) -->

---
<style scoped>
{
  font-size: 30px
}
</style>

#### LRU algorithm does not exist Belady phenomenon

![ w:1100 ]( figs/belady-lru.png )

Does clock/improved clock page replacement have the Belady phenomenon?
Why does the LRU page replacement algorithm not have the Belady phenomenon?

---

#### Comparison of LRU, FIFO and Clock

- LRU algorithm and FIFO are essentially first-in first-out ideas
- LRU is sorted by the most recent access time of the page
- LRU needs to dynamically adjust the order
- FIFO is sorted by the time the pages entered memory
- FIFO page entry time is fixed

---

#### Comparison of LRU, FIFO and Clock

- LRU can degenerate into FIFO
  - If the page has not been accessed after entering the memory , the latest access time is the same as the time when it entered the memory
  - For example: allocate 3 physical pages to the process, the access sequence of logical pages is 1, 2, 3, 4, 5, 6, 1, 2, 3...

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Comparison of LRU, FIFO and Clock

- The performance of the LRU algorithm is better, but the system overhead is larger
- The system overhead of FIFO algorithm is small, and the Belady phenomenon will occur
- Clock algorithm is their **compromise**
  - When the page is accessed, the order of the page in the linked list is not dynamically adjusted, only marked
  - When a page is missing, move it to the end of the linked list
  - For **not visited** pages, Clock and LRU algorithms perform as well
  - For **visited** pages, the Clock algorithm cannot record the exact access sequence, while the LRU algorithm can

---

### Summary

1. The basic concept of page replacement algorithm
2. Optimal page replacement algorithm (OPT, optimal)
3. First in first out page replacement algorithm (FIFO)
4. The least recently used page replacement algorithm (LRU, Least Recently Used)
5. Clock page replacement algorithm (Clock)
6. Improved clock page replacement algorithm
7. Least Frequently Used Algorithm (LFU, Least Frequently Used)
8. The Belady phenomenon









