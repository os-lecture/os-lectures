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

## Section 4 Linux CFS Scheduling
Completely Fair Scheduler (CFS, Completely Fair Scheduler)

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Principle of CFS
2. Implementation of CFS

---
<style scoped>
{
  font-size: 30px
}
</style>
#### CFS Background
<!-- 10,000-character long text, hammer it! Demystifying the Linux Process Scheduler https://www.eet-china.com/mp/a111242.html -->
- Both O(1) and O(n) divide CPU resources into time slices
     - Using a fixed quota allocation mechanism, the available time slice of each scheduling cycle is determined
     - The end of the scheduling period is reassigned
- The O(1) scheduler is essentially the idea of the MLFQ algorithm
     - Insufficient: O(1) scheduler's response to process interactivity is not timely
- Need
     - Judging whether it is IO-intensive or CPU-intensive according to the running status of the process, and then do priority rewards and penalties
     - There are errors in this speculation itself, the more complex the scene, the more difficult it is to judge


---
#### CFS Background

CFS scheduling algorithm proposed and implemented by Hungarian Ingo Molnar
- He is also the proposer of the O(1) scheduling algorithm

![bg right:40% 100%](figs/ingo-molnar.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### The idea of CFS
<!-- - CFS does not calculate priority, but determines who will schedule by calculating the CPU time consumed by the process (normalized virtual CPU time). So as to achieve the so-called fairness. -->
- Abandon fixed time slice allocation and adopt **dynamic time slice allocation**
- The time that a process can occupy in each scheduling is related to the total number of processes, total CPU time, process weight, etc., and the value of each scheduling cycle may be different
- Each process has a nice value, indicating its static priority

![bg right 90%](figs/prio-to-weight.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### The idea of CFS
- **Treat the CPU as a resource**, and record the use of this resource by each process
     - When scheduling, the scheduler always chooses the process that consumes the least resources to run (**fair distribution**)
- Since the work of some processes is more important than others, this absolute fairness is sometimes an unfair one
     - **Allocate CPU resources according to weight**
![bg right:47% 90%](figs/prio-to-weight.png)

---
#### CFS process running time dynamic allocation
- Allocate running time according to the priority weight of each process
     - The greater the weight of the process, the more running time it gets
`Running time allocated to a process = scheduling period * process weight / sum of all process weights`
- Scheduling cycle
     - The time to schedule all processes in the TASK_RUNNING state

<!-- ![bg right:40% 100%](figs/prio-to-weight.png) -->


---
<!-- CFS (Completely Fair Scheduler) https://www.jianshu.com/p/1da5cfd5cee4 -->
#### Relative Fairness of CFS
- There are two processes A and B in the system, the weights are 1 and 2 respectively, and the scheduling period is set to 30ms,
- CPU time for A is: 30ms * (1/(1+2)) = 10ms
- CPU time for B is: 30ms * (2/(1+2)) = 20ms
- Out of these 30ms A will run for 10ms and B will run for 20ms

Their runtimes are not the same. How is fairness reflected?

<!-- ![bg right 100%](figs/rbtree.png) -->



---

#### CFS virtual time vruntime

* Virtual runtime (vruntime): records the time the process has been running
     * vruntime increases or decreases the running time according to the weight of the process.
`vruntime = actual running time * 1024 / process weight`
     * 1024 is the weight of the process whose nice is 0, the code is NICE_0_LOAD
     * All processes use the weight 1024 of the process with nice as 0 as a benchmark to calculate their own vruntime increase speed

---

#### CFS virtual time vruntime

Taking the above two processes A and B as an example, the weight of B is twice that of A, so the vruntime increase rate of B is only half of that of A.
```
vruntime = (scheduling period * process weight / total weight of all processes) * 1024 / process weight
          = scheduling period * 1024 / total weight of all processes
```
Although the weights of the processes are different, their vruntime growth rates should be the same, regardless of the weight.

<!-- O(n), O(1) and CFS schedulers http://www.wowotech.net/process_management/scheduler-history.html

Virtual runtime = (physical runtime) X (weight of nice value 0) / weight of the process

Through the above formula, we construct a virtual world. The two-dimensional (load weight, physical runtime) physical world becomes a one-dimensional virtual runtime virtual world. In this virtual world, the vruntime of each process can be compared to determine its position in the red-black tree, and the fairness of the CFS scheduler is to maintain the fairness of the vruntime in the virtual world, that is, the vruntime of each process is equal.

-->

---
#### CFS virtual time calculation

From a macro perspective, the vruntime growth rate of all processes should be promoted at the same time, and this vruntime can be used to select the running process.

* The smaller vruntime value of the process means that it used to occupy the cpu for a shorter time and was treated "unfairly", so it is the next running process.
* This can not only select processes fairly, but also ensure that high-priority processes get more running time.





---
#### CFS virtual time calculation example

CFS makes the vruntime of each scheduling entity (process or process group) catch up with each other, and the vruntime of each scheduling entity increases at different speeds. The greater the weight, the slower the increase, so that more CPU execution time can be obtained.

      A has 6 time slices per cycle, B has 3 time slices per cycle, and C has 2 time slices per cycle
      vruntime:
      A: 0 6 6 6 6 6 6 12 12 12 12 12 12
      B: 0 0 3 3 3 6 6 6 9 9 9 12 12
      C: 0 0 0 2 4 4 6 6 6 8 10 10 12
      Scheduling: A B C C B C A B C C B C


<!-- ![bg right 100%](figs/rbtree.png) -->


---

**Outline**

1. Principle of CFS
### 2. Implementation of CFS

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Red-black tree: process vruntime data structure in CFS
- Linux uses a red-black tree to record the vruntime of each process
     - On multi-core systems, one red-black tree per core
     - When scheduling, select the process with the smallest vruntime from the red-black tree to run
![bg right:53% 90%](figs/rbtree.png)

---

#### Process weight for CFS
- The weight is determined by the nice value, and the weight corresponds to the nice value of the process one by one
     - The larger the nice value, the lower the weight
- Convert by global array prio_to_weight
![bg right:48% 90%](figs/prio-to-weight.png)


---
<style scoped>
{
  font-size: 32px
}
</style>
#### How to set the vruntime of the newly created process in CFS?

- If the initial value of vruntime of the new process is 0, which is much smaller than the value of the old process, then it will maintain the advantage of preempting the CPU for a long time, and the old process will starve to death, which is obviously unfair.
![bg right:46% 90%](figs/rbtree.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Vruntime settings for newly created processes in CFS

- Each CPU's run queue cfs_rq maintains a **min_vruntime field**
     - Record the minimum value of vruntime for all processes in the run queue
- The new process's initial vruntime value is set to the min_vruntime of the run queue it is in
     - stay within a reasonable distance from the old process
![bg right:41% 90%](figs/rbtree.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Does the vruntime of a sleeping process in CFS stay the same all the time?

If the vruntime of the dormant process remains the same, and the vruntime of other running processes keeps advancing, then when the dormant process finally wakes up, its vruntime is much smaller than others, giving it the advantage of seizing the CPU for a long time, while other processes Going to starve to death.
![bg right:47% 100%](figs/rbtree.png)


---

#### Vruntime of sleeping process in CFS

- Reset the vruntime value when the dormant process is woken up, based on the min_vruntime value, give some compensation, but not too much compensation.
![bg right:45% 90%](figs/rbtree.png)


---
<style scoped>
{
  font-size: 30px
}
</style>
#### Will a dormant process in CFS seize the CPU immediately when it wakes up?

- It is a high probability event that the dormant process has the ability to **preempt** the CPU when it wakes up. This is also the original intention of the CFS scheduling algorithm, that is, to ensure the response speed of the interactive process. **Interactive processes** wait for user input frequently. sleep.
![bg right:45% 100%](figs/rbtree.png)

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Will a dormant process in CFS seize the CPU immediately when it wakes up?

- **Processes that are actively sleeping** will also be compensated when they wake up. Such processes often do not require fast response. They will also wake up and preempt each time, which may cause other more important application processes Being preempted, detrimental to overall performance.
- The WAKEUP_PREEMPT bit of sched_features indicates that the wakeup preemption feature is disabled. The process that has just been woken up **does not immediately preempt** the running process, but waits until the running process runs out of time slices

![bg right:30% 100%](figs/rbtree.png)


---

#### Will the vruntime change when the process in CFS migrates between CPUs?
- On a multi-CPU system, different CPUs have different loads, some CPUs are busier, and each CPU has its own run queue, and the vruntime of the processes in each queue also **runs faster and sometimes faster Slow**, the min_vruntime value of each CPU running queue will be different
![bg right:40% 90%](figs/smp-min-vruntime.png)



---

#### Process Migration in CFS

- When a process **comes out** of a CPU's run queue, its vruntime is **subtracted** from the min_vruntime value of the queue;
- When a process **joins** another CPU's run queue, its vruntime is **added** to the queue's min_vruntime value.

![bg right:45% 90%](figs/smp-min-vruntime.png)

---
#### CFS vruntime overflow problem

- Type usigned long for vruntime
- The virtual time of the process is an increasing positive value, so it will not be negative, but it has its upper limit, which is the maximum value that can be represented by unsigned long
- If it overflows, then it will roll back from 0, if so, what will happen?

![bg right:40% 100%](figs/rbtree.png)

---

#### CFS vruntime overflow example
```C
unsigned char a = 251;
unsigned char b = 254;
b += 5;
//b overflows, causing a > b, should b = a + 8
//How to achieve the real result? Change to the following:
unsigned char a = 251;
unsigned char b = 254;
b += 5;
signed char c = a - 250,
signed char d = b - 250;
//Here to judge the size of c and d
```
![bg right:54% 90%](figs/rbtree.png)

---

#### [Modularization] of the Linux scheduler (http://www.wowotech.net/process_management/scheduler-history.html)

![w:650](figs/Linux-scheduler-module.png)

---
### References
- https://www.eet-china.com/mp/a111242.html
- https://www.jianshu.com/p/1da5cfd5cee4
- https://developer.ibm.com/tutorials/l-completely-fair-scheduler/
- http://www.wowotech.net/process_management/scheduler-history.html

---
<style scoped>
{
  font-size: 32px
}
</style>
### Course experiment three process and process management

* Chapter 5: Process and process management -> chapter5 exercises ->
     * [rCore](https://learningos.github.io/rCore-Tutorial-Guide-2022A/chapter5/4exercise.html)
     * [uCore](https://learningos.github.io/uCore-Tutorial-Guide-2022A/chapter5/4exercise.html)
* Experimental tasks
     * spawn system call
     * stride scheduling algorithm
* Experiment submission requirements
     * The 11th day after the task is assigned (November 13, 2022);