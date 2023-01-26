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

# Lecture 7 Process Management and Single Processor Scheduling
## Section 2 Uniprocessor Scheduling

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Processor scheduling concept
   - Timing and strategy for processor scheduling
   - Criteria for comparing scheduling algorithms
2. Scheduling algorithm

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Time division multiplexing of CPU resources

- Process switching: switching of current occupants of CPU resources
   - Save the execution context (CPU state) of the current process in the PCB
   - restore the execution context of the next process

- Processor scheduling
    - Pick the next CPU-intensive process from the ready queue
    - Select the CPU resources available to the ready process from multiple available CPUs
- Scheduler: Kernel function that picks ready processes
- Scheduling strategy
     - On what basis are the processes selected?


---
#### Scheduling timing

- The conditions under which the kernel executes the schedule
    - Process switches from running state to waiting/ready state
    - the process was terminated
- Non-preemptive systems
   - When the current process actively relinquishes the CPU
- Preemptable system
   - When the interrupt request is serviced by the service routine

---

#### Scheduling strategy

Determines how to select the next process from the ready queue for execution
- problem to be solved
    - By what criteria to choose?
    - Which process in the ready queue is picked?
- Scheduling Algorithm
    - Scheduling strategy implemented in kernel scheduling
- Criteria for comparing scheduling algorithms
    - Which strategy/algorithm is better?
 
---
#### Processor Resource Usage Patterns
- Process alternates between CPU calculations and I/O operations
    - Each scheduler decides which work to hand over to the CPU for the next CPU calculation
    - Under the time slice mechanism, the process may be forced to give up the CPU before ending the current CPU calculation

  ![w:650](figs/cpu-usage.png)

 
---

**Outline**

1. The concept of processor scheduling
   - Timing and strategy for processor scheduling
### Criteria for comparing scheduling algorithms
2. Scheduling algorithm

---

#### Criteria for comparing scheduling algorithms
- CPU usage: the percentage of time the CPU is busy
- Throughput: the number of processes completed per unit of time
- Turnaround time: the total time the process takes from initialization to completion (including waiting)
- Ready wait time: the total time that the ready process is in the ready queue
- Response time: the total time taken from submitting a request to generating a response
- Fairness: processes occupy the same resources, such as CPU time, etc.


 
---
<style scoped>
{
  font-size: 30px
}
</style>

#### Comparing throughput and latency criteria for scheduling algorithms
- Scheduling Algorithm Requirements: Desire for "faster" service
- What is faster?
   - High bandwidth when transferring files, high throughput for scheduling algorithms
   - Low latency when playing games, low response latency for scheduling algorithms
   - These two factors influence each other
- Analogy with water pipes
   - Low latency: when you drink water, you want the water to flow out as soon as you open the tap
   - High bandwidth: When filling a pool, expect a lot of water to flow from the faucet at the same time, and don't mind the lag

 
---
<style scoped>
{
  font-size: 30px
}
</style>

#### Comparing response time criteria for scheduling algorithms
- Reduced response time
   - Process the user's input request in a timely manner, and feedback the output to the user as soon as possible
- Reduce fluctuations in average response time
   - In interactive systems, predictability is more important than high variance low average
- Low-latency scheduling improves user interaction experience
   - If the cursor on the screen does not move when the mouse is moved, the user may restart the computer
- Response time is the calculation delay of the operating system

 
---
#### Comparing throughput criteria for scheduling algorithms
- increase throughput
    - Reduced overhead (OS overhead, context switching)
    - Efficient utilization of system resources (CPU, I/O devices)
- Reduced ready wait time
    - Reduced waiting time for each ready process
- The operating system needs to guarantee that throughput is not affected by user interaction
   - Even if there are many interactive tasks
- Throughput is the computing bandwidth of the operating system

 
---
#### Comparing fairness criteria for scheduling algorithms

Is it fair when one user runs more processes than other users? what to do?

- Definition of fairness
   - Guarantee that each process takes the same amount of CPU time
   - Guarantee that the ready wait time of each process is the same
- Fairness usually increases average response time

 
---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. The concept of processor scheduling
### 2. Scheduling algorithm
- First come first serve algorithm FCFS, short job first algorithm SJF
- The shortest remaining time algorithm SRT, the highest response ratio priority algorithm HRRN
- Time slice rotation algorithm RR
- Multi-level queue scheduling algorithm MQ, multi-level feedback queue algorithm MLFQ
- Fair Share Scheduling Algorithm FSS


---

#### First come first serve scheduling algorithm FCFS

FCFS: First Come, First Served
- Arranged according to the order in which processes enter the ready state
- When a process enters a wait or end state, the next process in the ready queue occupies the CPU
- indicators
    - Turnaround time of FCFS algorithm
  
---

#### First come first serve scheduling algorithm example

- Example: 3 processes with calculation times of 12,3,3

![w:800](figs/sched-fifo.png)

   
---

#### Characteristics of First Come First Serve Scheduling Algorithm

- Pros: Simple
- Shortcoming
   - Average wait time fluctuates widely
   - short jobs/tasks/processes may be queued behind long ones
   - Low utilization of I/O resources and CPU resources
      - When CPU-intensive processes cause I/O devices to be idle,
I/O-bound processes also wait
  
---

#### Short Job First Scheduling Algorithm SJF

Short Job First
- Select the job/process with the shortest execution time in the ready queue to occupy the CPU and enter the running state
- The ready queue is sorted by expected execution time

  ![w:800](figs/sched-sjf.png)

---

#### Characteristics of Short Job First Scheduling Algorithm

**With the best average turnaround time**

Is it possible to modify the order of job/process execution to reduce the average wait time?
  ![w:800](figs/sched-sjf-best.png)


  ---

#### Characteristics of Short Job First Scheduling Algorithm

- May cause hunger
   - Continuous streams of short jobs/processes starve long jobs/processes of CPU resources

- Need to predict the future
   - How to estimate the duration of the next CPU calculation?
   - Simple solution: ask the user
      - Kill the corresponding process if the user cheats
      - Users don't know what to do?
---

#### Execution Time Estimation of the Shortest Job First Algorithm

- Use historical execution time to predict future execution time

$\tau_{n+1} = \alpha t_n+(1-\alpha) \tau_n, where 0\le \alpha \le 1$
$t_n$ -- the nth CPU calculation time
$\tau_{n+1}$ -- CPU computing time estimate for the n+1th time

$\tau_{n+1} = \alpha t_n+(1-\alpha) \alpha t_{n-1} + (1-\alpha) (1-\alpha) \alpha t_{n-2} + .. .$


---

#### Execution Time Estimation of the Shortest Job First Algorithm

- Execution time estimate
  ![w:800](figs/sched-sjf-time.png)



---

#### The shortest remaining time algorithm SRT

Shortest Remaining Time, SRT

- SRT supports the preemptive scheduling mechanism, that is, if a new process is ready, and the service time of the new process is less than the remaining time of the current process, it will transfer to the new process for execution.

---

#### Highest Response Ratio Priority Algorithm HRRN

Highest Response Ratio Next, HRRN

- High response ratio priority scheduling algorithm is mainly used for job scheduling
- This algorithm is a comprehensive balance of FCFS scheduling algorithm and SJF scheduling algorithm, taking into account the waiting time and estimated running time of each job
- When scheduling jobs, first calculate the response ratio of each job in the backup job queue, and select the job with the highest response ratio to run.

---

#### Highest Response Ratio Priority Algorithm HRRN

- Select the process with the highest response ratio R value in the ready queue

$R=(w+s)/s$
w: ready waiting time (waiting time)
s: execution time (service time)

- Improved on the basis of short job first algorithm
- Keep an eye on the waiting time of the process
- prevent indefinite postponement

---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. The concept of processor scheduling
2. Scheduling algorithm
- First come first serve algorithm FCFS, short job first algorithm SJF
- The shortest remaining time algorithm SRT, the highest response ratio priority algorithm HRRN
### Time slice rotation algorithm RR
- Multi-level queue scheduling algorithm MQ, multi-level feedback queue algorithm MLFQ
- Fair Share Scheduling Algorithm FSS

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Time slice rotation algorithm RR

RR, Round-Robin

- Time slice
   - The basic unit of time for allocating processor resources
- Algorithm ideas
   - At the end of the time slice, switch to the next ready process according to the FCFS algorithm
   - Execute a time slice q every (n – 1) time slice process
  ![w:700](figs/sched-rr-1.png)

  ---

#### Time slice rotation algorithm example

  ![w:700](figs/sched-rr-2.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### The time slice length parameter of the time slice rotation algorithm

- RR algorithm overhead: additional context switching
- time slice too large
   - The waiting time is too long, and the limit case degenerates into FCFS
- Time slice is too small
   - Responsive, but produces lots of context switches
   - A lot of context switching overhead affects system throughput
- Time slice length selection target
   - Choose an appropriate time slice length
   - Rule of thumb: keep context switch overhead under 1%

---

#### Comparing FCFS and RR

![w:800](figs/sched-rr-3.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. The concept of processor scheduling
2. Scheduling algorithm
- First come first serve algorithm FCFS, short job first algorithm SJF
- The shortest remaining time algorithm SRT, the highest response ratio priority algorithm HRRN
- Time slice rotation algorithm RR
### Multi-level queue scheduling algorithm MQ, multi-level feedback queue algorithm MLFQ
- Fair Share Scheduling Algorithm FSS

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Multi-level queue scheduling algorithm MQ

MQ, MultiQueue
- The ready queue is divided into multiple independent subqueues
    - Such as: foreground process (interactive) subqueue, background process (batch processing) subqueue
    - Processes of the same priority belong to a queue and cannot cross queues
- Each queue has its own scheduling policy
    - Such as: foreground process – RR, background process – RR/FCFS with a large time slice
 
  - Rule 1: If A's priority > B's priority, run A (do not run B).
  - Rule 2: If A's priority = B's priority, run A and B alternately.
---

#### Multi-level queue scheduling algorithm MQ

- Scheduling between queues
   - fixed priority
     - Handle foreground (interactive) processes first, then background processes
     - May cause hunger
   - Time slice rotation
     - Each queue gets a certain amount of CPU time that can schedule its processes
     - For example: 80% CPU time is used for foreground processes, 20% CPU time is used for background processes


---

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

MLFQ, Multi-Level Feedback Queue
- In 1962, MIT professor Corbato first proposed a multi-level feedback queue, which was applied to a compatible time-sharing system (CTSS-Compatible Time-Sharing System)
- Solve two problems
   - How to optimize turnaround time without knowing how long the job will run
   - How to reduce the response time and give interactive users a good interactive experience


---

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

- Key question: how to schedule without complete knowledge?
    - How do you build a scheduler that reduces both response time and turnaround time when the length of the process work is unknown?
- Inspiration: Learning from History
    - Predict the future with historical experience
- Inherit the scheduling rules of Multi Queue
    - If A's priority > B's priority, run A (do not run B)
    - If A's priority = B's priority, round robin/FIFO runs A and B
---

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

- Key question: how to schedule without complete knowledge?
    - How do you build a scheduler that reduces both response time and turnaround time when the length of the process work is unknown?
- Inspiration: Learning from History
    - Predict the future with historical experience
- Inherit the scheduling rules of Multi Queue
    - If A's priority > B's priority, run A (do not run B)
    - If A's priority = B's priority, round robin/FIFO runs A and B
---
<style scoped>
{
  font-size: 30px
}
</style>

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

Basic scheduling rules
  - When a job enters the system, it is placed at the highest priority (top queue)
  - If the process is not completed in the current time slice, it will drop to the next priority
  - If the job actively releases the CPU within its time slice, the priority remains the same
  - Time slice size increases with priority level
  ![w:500](figs/sched-mlfq.png)

 
---

#### MLFQ scheduling example for three priority queues

- CPU-intensive processes enter the highest priority queue first;
- After executing the 1ms time slice, the scheduler will reduce the priority of the process by 1 and enter the next highest priority queue;
- After executing the 2ms time slice, enter the lowest priority queue of the system, stay there, and execute according to the 4ms time slice.

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

- Characteristics of the MLFQ algorithm
    - The priority of CPU-intensive processes drops quickly
    - I/O intensive processes stay at high priority

- Potential problems
   - CPU intensive processes will starve
   - Malicious processes will find ways to stay in high priority
  ![bg right:40% 100%](figs/sched-mlfq.png)

 
---

<style scoped>
{
  font-size: 32px
}
</style>

#### Multi-Level Feedback Queue Scheduling Algorithm MLFQ

Basic scheduling rules
  - If A's priority > B's priority, run A (do not run B)
  - If A's priority = B's priority, round robin/FIFO runs A and B
  - When a job enters the system, it is placed at the highest priority (top queue)
  - Once a job has used up its time quota in a certain layer (no matter how many times the CPU is actively given up in the middle), reduce its priority (move to the lower queue)
  - After a period of time S, rejoin all jobs in the system to the highest priority queue

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Fair Share Scheduling Algorithm FSS

FSS, Fair Share Scheduling
- Control user access to system resources
    - Multiple processes owned by different users
    - Allocate resources according to user priority
    - Ensure that unimportant users cannot monopolize resources
    - Unused resources are allocated proportionally
  
  ![bg right:40% 100%](figs/sched-fss.png)

  
---
<style scoped>
{
  font-size: 32px
}
</style>

#### Characteristics of the scheduling algorithm

- First come first serve algorithm
    - Poor average wait time
- Shortest job first algorithm
    - Minimal average turnaround time
    - Requires accurate prediction of computation time
    - No preemption allowed; may cause starvation
- Minimum remaining time algorithm
    - Improvements to the short job first algorithm, allowing preemption
    - May cause hunger
  
---
<style scoped>
{
  font-size: 32px
}
</style>

#### Characteristics of the scheduling algorithm

- Highest Response Ratio Priority Algorithm
    - Based on short job priority scheduling, non-preemptive
    - Consider both the wait time and the estimated run time of each job
- Time slice rotation algorithm
    - Fair, but average wait time is poor
- Multi-level feedback queue algorithm
    - Integration of multiple algorithms
- Fair share scheduling algorithm
    - Fairness is the first element

---

### Summary

1. The concept of processor scheduling
- Timing and strategy of processor scheduling, criteria for comparing scheduling algorithms
2. Scheduling algorithm
- FCFS, SJF, SRT, HRRN
- RR
- MQ, MLFQ, FSS