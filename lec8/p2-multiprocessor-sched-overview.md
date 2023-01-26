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

## Section 2 Overview of Multiprocessor Scheduling


<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Single queue multiprocessor scheduling SQMS
2. Multi-queue multi-processor scheduling MQMS

---

#### Single Queue Multiprocessor Scheduling
Single Queue Multiprocessor Scheduling, SQMS
- Reuse the basic architecture under uniprocessor scheduling
- All processes that need to be scheduled are put into **a queue**
![w:800](figs/sqms.png)


---
#### Characteristics of Single Queue Multiprocessor Scheduling
- Lack of scalability
- Weak cache affinity
![w:800](figs/sqms.png)

---
#### Affinity and load balancing of multiprocessor scheduling
Run processes on the same CPU as much as possible. While maintaining the affinity of some processes, it may be necessary to sacrifice the affinity of other processes to achieve load balancing.
![bg right:50% 90%](figs/sqms-cache-affinity.png)

---

**Outline**

1. Single queue multiprocessor scheduling SQMS
### 2. Multi-queue multi-processor scheduling MQMS

---

#### Multi-queue multi-processor scheduling
Multi-Queue MultiprocessorScheduling, MQMS
- The basic scheduling framework contains **multiple scheduling queues**, and each queue can use different scheduling rules.
- When a process enters the system, it is put into a dispatch queue according to some heuristic rules.
- **Each CPU schedules independently** to avoid data sharing and synchronization problems in single queue mode.
![w:800](figs/multi-queue.png)
![w:800](figs/mqms.png)

---
#### Characteristics of multi-queue multi-processor scheduling
- It is **scalable**: the number of queues will increase with the increase of CPU, so the overhead of locks and cache contention is not a big problem.
- Has good **cache affinity**: all processes stay on a fixed CPU and thus make good use of cached data.
![w:800](figs/multi-queue.png)
![w:800](figs/mqms.png)


---
#### Uneven load of multi-queue multi-processor scheduling
- Assume 4 processes and 2 CPUs; the queues all execute the round-robin scheduling strategy; process C is executed
- A gets twice as much CPU time as B and D

![w:1000](figs/mqms-problem-1.png)
![w:1000](figs/mqms-problem-2.png)


---

#### How to solve the uneven load of MQMS?

- Assume 4 processes and 2 CPUs; each queue executes a round-robin scheduling strategy; both A and C are executed, and only B and D are in the system
   - CPU1 is busy
   - CPU0 is idle

![w:900](figs/mqms-problem-3.png)
![w:900](figs/mqms-problem-4.png)



---
#### Process Migration (migration)
- Load balancing can be achieved through cross-CPU migration of processes.
   - Situation: CPU0 is idle, CPU1 has some processes.
   - Migrate: Migrate B or D to CPU0

![w:900](figs/mqms-problem-3.png)
![w:900](figs/mqms-problem-4.png)



---
#### How does MQMS determine the timing of process migration?

- Situation: A is left alone on CPU 0, B and D run alternately on CPU 1
- Migration: **constantly migrating and switching** one or more processes

![w:900](figs/mqms-problem-5.png)
![w:900](figs/mqms-problem-6.png)



---
<style scoped>
{
  font-size: 32px
}
</style>

#### MQMS work stealing (work stealing)
- The (source) queue with fewer processes periodically "peeks" at other (destination) queues to see if there are more processes than its own
- Load balancing by "stealing" one or more processes from the target queue if the target queue is (significantly) fuller than the source queue.


![w:900](figs/mqms-problem-5.png)
![w:900](figs/mqms-problem-6.png)

---
#### Queue check interval for work stealing
- If you frequently check other queues, it will bring high overhead and poor scalability
- If the check interval is too long, it may cause serious load imbalance
![w:900](figs/mqms-problem-5.png)
![w:900](figs/mqms-problem-6.png)