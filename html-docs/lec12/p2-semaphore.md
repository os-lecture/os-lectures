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

# Lecture 12 Synchronization and mutual exclusion

## Section 2 Semaphore

---
<style scoped>
{
  font-size: 30px
}
</style>

### Semaphore
- A semaphore is a method provided by the operating system to coordinate access to shared resources
- Proposed by Dijkstra in the 1960s
- Primary synchronization mechanism for early operating systems
![w:600](figs/basic-syncmutex.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

### Semaphore
- A semaphore is an abstract data type consisting of an integer (sem) variable and two atomic operations
    - P(): Prolaag Dutch: try to reduce
       - $sem = sem - 1$
       - If sem<0, enter waiting, otherwise continue
    - V(): Verhoog Dutch: increase
       - $sem = sem + 1$
       - Like $sem \le 0$, wake up a waiting
![bg right:40% 100%](figs/sema-train.png)

---
<style scoped>
{
  font-size: 28px
}
</style>

### Semaphore
- Semaphores are protected integer variables
    - After initialization, it can only be modified by P() and V() operations
    - Guaranteed by the **operating system**, PV operations are atomic
- P() may block, V() will not block
- Semaphores are usually assumed to be "fair"
    - The thread will not be blocked indefinitely in the P() operation
    - Assume semaphore waits are queued on a first-in-first-out basis

Can spinlocks implement first-in-first-out?

![bg right:30% 100%](figs/sema-train.png)


---
### Semaphore
Conceptual realization of semaphore
![w:1200](figs/semaphore-impl.png)


---
### Semaphore
There are two types of semaphores
- Binary semaphore: the number of resources is 0 or 1
- Counting semaphore: the number of resources is any non-negative value
- Both are equivalent: based on one the other can be implemented

The use of semaphores
- **Mutual Exclusive Access** and **Conditional Synchronization**

![bg right:40% 100%](figs/sema-train.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

### Semaphore
Example of Mutual Exclusion
- Each critical section sets a semaphore with an initial value of 1
- P() operation and V() operation need to be used in pairs
    - The P() operation guarantees exclusive access to the resource
    - The V() operation releases resources after use
    - The sequence of PV operations cannot be wrong, repeated or omitted

![bg right:40% 100%](figs/semaphore-use-1.png)
 


---
### Semaphore
Example of conditional synchronization
- Each condition synchronously sets a semaphore, whose initial value is 0

![bg right:50% 100%](figs/semaphore-use-2.png)



---
### Semaphore
Example of producer-consumer problem
- Producer-consumer problem description for bounded buffers
    - One or more producers place data in a buffer after producing it
    - A single consumer fetches data from the buffer for processing
    - Only one producer or consumer can access the buffer at any time
![w:700](figs/semaphore-use-3.png)


---
<style scoped>
{
  font-size: 28px
}
</style>

### Semaphore
Example of producer-consumer problem
- Problem analysis
    - Only one thread can operate the buffer at any time (mutually exclusive access)
    - When the buffer is empty, the consumer must wait for the producer (conditional synchronization)
    - When the buffer is full, the producer must wait for the consumer (conditional synchronization)
- Describe each constraint with a semaphore
    - Binary semaphore mutex
    - Count semaphore fullBuffers
    - Count semaphore emptyBuffers

---
### Semaphore
Examples of producer-consumer problems: Does the order of P and V operations matter?
![w:900](figs/semaphore-use-4.png)


---
### Semaphore
- Difficult to read/develop code
- Easy to make mistake
    - Use a semaphore that is already in use
    - Forgetting to release the semaphore
    - Can't avoid deadlock problem
    - High requirements for programmers
![bg right:40% 100%](figs/semaphore-use-4.png)