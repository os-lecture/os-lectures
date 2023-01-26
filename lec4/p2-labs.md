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

# Lecture 4 Multiprogramming and time-sharing multitasking
## Section 2 Practice: Multiprogramming and time-sharing multitasking operating system
<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---
**Outline**

### 1. Experimental objectives and steps
- Experiment objectives
- Practical steps
2. Multi-channel batch processing operating system design
3. Application Design
4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
6. Coelophysis OS: time-sharing multitasking OS

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Experiment Objectives

![bg right:43% 90%](figs/multiprog-os-detail.png)

- MultiprogOS target
     - Further improve the overall performance and efficiency of multiple applications in the system
- BatchOS target
     - Isolate APP from OS to improve system security and efficiency
- LibOS target
     - Isolate applications from hardware, simplifying the difficulty and complexity of applications accessing hardware

---

#### Experiment Requirements
- Understand
     - Cooperative scheduling and preemptive scheduling
     - Tasks and task switching
- Write
     - Multiprogramming operating system
     - Time-sharing multitasking operating system

<!-- Sawtooth Archosaurus Coelophysis -->

![bg right:45% 70%](figs/ch3-oses.png)

---
#### Structure of a multiprogramming system

![bg 55%](figs/multiprog-os-detail.png)

---
#### General idea
- Compilation: The application program and the kernel are compiled independently and merged into one image
- Compilation: applications require their respective start addresses
- Construction: system call service request interface, process management and initialization
- Structure: process control block, process context/state management
- Run: privilege level switching, process and OS switching
- Running: The process realizes active/passive switching through system calls/interrupts

---
#### History background

- 1961 British Leo III computer
- Supports **loading** multiple different programs in computer memory and running **sequentially** from the first one
<!-- - J. Lyons & Co. for commercial transactions

J. Lyons & Co. is a British restaurant chain, food manufacturing and hotel group founded in 1884. -->

<!-- https://baike.baidu.com/item/EDSAC/7639053
Electronic Delay Storage Automatic Calculator (English: Electronic Delay Storage Automatic Calculator, EDSAC) is an early British computer. In 1946, Professor Maurice Wilkes of the Mathematics Laboratory of the University of Cambridge and his team were inspired by von Neumann's First Draft of a Report on the EDVAC, and designed and built EDSAC based on EDVAC. It was officially put into operation on May 6, 1949. It is the world's first stored-program electronic computer that actually runs.
It was EDSAC that also encountered difficulties in project implementation: not technology, but lack of funds. At the crucial moment, Wilkes successfully persuaded J. Lyons & Co. ． The boss invested in the project and finally brought the plan back to life. On May 6, 1949, EDSAC was successfully tested for the first time. It read a program to generate a square table from the tape and executed it, and printed the result correctly. In return for the investment, LyOHS obtained the right to mass-produce EDSAC, which is the LEO computer (Lyons Electronic Office) that was officially put on the market in 1951, which is generally considered to be the first commercialized computer model in the world, so This has also become an interesting fact in the history of computer development: the first manufacturer to produce a commercial computer turned out to be a bakery. Lyons company later became part of the famous "International Computer Co., Ltd.", or ICL, in the UK.
-->

![bg right 100%](figs/multiprog-os.png)

---
**Outline**

1. Experimental objectives and steps
- Experiment objectives
- **Practical steps**
2. Multi-channel batch processing operating system design
3. Application Design
4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
6. Coelophysis OS: time-sharing multitasking OS

---

#### Practical steps (based on BatchOS)
- Modify the APP link script (custom start address)
- Load & execute application
- Switch tasks

![bg right 100%](figs/multiprog-os.png)

---

#### Three applications are executed alternately
```
git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
cd rCore-Tutorial-v3
git checkout ch3-coop
```
Contains three applications, everyone humbly **executes alternately**
```
user/src/bin/
├── 00write_a.rs # Display AAAAAAAAAA string 5 times
├── 01write_b.rs # Display BBBBBBBBBB string twice
└── 02write_c.rs # Display CCCCCCCCCC string 3 times
```

---

#### operation result
```
[RustSBI output]
[kernel] Hello, world!
AAAAAAAAAAA [1/5]
BBBBBBBBBB [1/2]
....
CCCCCCCCCC [2/3]
AAAAAAAAAAA [3/5]
Test write_b OK!
[kernel] Application exited with code 0
CCCCCCCCCC [3/3]
...
[kernel] Application exited with code 0
[kernel] Panicked at src/task/mod.rs:106 All applications completed!
```


---
**Outline**

1. Experimental objectives and steps
### 2. Multi-channel batch processing operating system design
3. Application Design
4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
6. Coelophysis OS: time-sharing multitasking OS

---

#### Software Architecture

![bg 55%](figs/multiprog-os-detail.png)

---

#### Code Structure: Application
Build application
```
└── user
     ├── build.py (new: use build.py to build applications so that the physical address ranges they occupy are disjoint)
     ├── Makefile (modification: use build.py to build the application)
     └── src (various applications)
```


---

#### Code structure: perfect task management function

Improve OS: ``Loader`` module loads and executes programs
```
├── os
│ └── src
│ ├── batch.rs (removed: the function is divided into two submodules, loader and task)
│ ├── config.rs (new: save some configurations of the kernel)
│ ├── loader.rs (new: load the application into the memory and manage it)
│ ├── main.rs (modification: the main function has been modified)
│ ├── syscall (modification: add some syscalls)
```

---

#### Code structure: process switching

Improve OS: `TaskManager` module manages/switches execution of programs
```
├── os
│ └── src
│ ├── task (new: task sub-module, mainly responsible for task management)
│ │ ├── context.rs (introduce Task context TaskContext)
│ │ ├── mod.rs (global task manager and interface provided to other modules)
│ │ ├── switch.rs (interprets the assembly code of task switching as the Rust interface __switch)
│ │ ├── switch.S (assembly code for task switching)
│ │ └── task.rs (definition of task control block TaskControlBlock and task status TaskStatus)
```

---

**Outline**

1. Experimental objectives and steps
2. Multi-channel batch processing operating system design
### 3. Application Design
4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
6. Coelophysis OS: time-sharing multitasking OS

---

#### Application Project Structure

No updates App names have numbers

```
user/src/bin/
├── 00write_a.rs # Display AAAAAAAAAA string 5 times
├── 01write_b.rs # Display BBBBBBBBBB string twice
└── 02write_c.rs # Display CCCCCCCCCC string 3 times
```
---

#### Application memory layout

- Since each application is loaded in a different location, the **``BASE_ADDRESS``** in their linker script linker.ld are all different.
- Write a script customization tool `build.py`, which customizes its own link script for each application
    - **``Application start address = base address + number number * 0x20000``**

---

#### Yield system call

```rust
//00write_a.rs
fn main() -> i32 {
     for i in 0..HEIGHT {
         for _ in 0..WIDTH {
             print!("A");
         }
         println!(" [{}/{}]", i + 1, HEIGHT);
         yield_(); //Give up the handler
     }
     println!("Test write_a OK!");
     0
}
```

---

#### Yield system call

- Apps are **mutually unaware**
- Application requires **actively relinquishes** processor
- needs to be implemented via a **new syscall**
   - **``const SYSCALL_YIELD: usize = 124;``**



---

#### Yield system call

``` Rust
const SYSCALL_YIELD: usize = 124;
pub fn sys_yield() -> isize {
     syscall(SYSCALL_YIELD, [0, 0, 0])
}
pub fn yield_() -> isize {
     sys_yield()
}
```


---
**Outline**

1. Experimental objectives and steps
2. Multi-channel batch processing operating system design
3. Application Design
### 4. Axolotl OS: support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
6. Coelophysis OS: time-sharing multitasking OS

---
#### Sawtooth OS: support application loading

The Permian "sawtooth salamander" operating system supports multiple applications residing in memory to form a multiprogramming operating system – Multiprog OS;
  
![bg right:50% 100%](figs/multiprog-os-detail.png)

---

#### Multi-Program Loading
- Apps are loaded differently
- All applications are loaded into memory at the time of kernel initialization
- To avoid overwriting, they naturally need to be **loaded at different physical addresses**


---

#### Multi-Program Loading

```Rust
fn get_base_i(app_id: usize) -> usize {
     APP_BASE_ADDRESS + app_id * APP_SIZE_LIMIT
}

let base_i = get_base_i(i);
// load app from data section to memory
let src = (app_start[i]..app_start[i + 1]);
let dst = (base_i.. base_i+src. len());
dst.copy_from_slice(src);
```


---

#### Execute program

- Execution timing
   - When the initial placement of the multiprogramming job is complete
   - When an application ends or an error occurs

- Implementation modalities
   - Call the run_next_app function to **switch** to the first/next application
  

---

#### Switch to the next program

   - Kernel mode to user mode
   - From user mode to kernel mode


---

#### Switch to the next program

   - Jump to entry point `entry(i)` of application number i for number i
   - Switch the used stack to the user stack stack(i)

![bg right:55% 90%](figs/kernel-stack.png)


---

#### execute program

Axolotl OS that supports **putting applications in memory** is now complete
![bg right:57% 95%](figs/jcy-multiprog-os-detail.png)

---
**Outline**

...

4. Sawtooth OS: Support application loading
### 5. Shichulong OS: Support multi-program cooperative scheduling
* Task switching
* Trap control flow switching
* Collaborative Scheduling
6. Coelophysis OS: time-sharing multitasking OS

---

#### Support multi-program cooperative scheduling

Cooperative multi-programming: the application **voluntarily gives up** the CPU and **switches** to another application to continue execution, thereby improving the overall execution efficiency of the system;

![bg right:50% 90%](figs/more-task-multiprog-os-detail.png)

---

#### Task switching

![bg 60%](figs/more-task-multiprog-os-detail.png)

---

#### Process

- **Process (Process)**: A **dynamic execution** process of a program with certain **independent functions** on a **data collection**. Also known as **Task (Task)**.
![bg right 100%](figs/task-multiprog-os-detail.png)


---

#### Time slice (slice)

- The execution segment or idle segment in a time segment during application execution is called "computing task slice" or "idle task slice", collectively referred to as **task slice** (task slice)
![bg right 100%](figs/task-multiprog-os-detail.png)



---
#### Task running status
   - Application execution within a time slice
     - running
     -ready

```rust
pub enum TaskStatus {
     UnInit,
     Ready,
     Running,
     Exited,
}
```
![bg right:50% 100%](figs/more-task-multiprog-os-detail.png)

---

#### Task switching
   - Switch from the execution process of one application to the execution process of another application
     - Pause the execution of an application (current task)
     - continue the execution of another application (next task)
![bg right:50% 100%](figs/more-task-multiprog-os-detail.png)


---

#### Task Context
- The **execution state (context)** of the application running at a certain moment
   - Execution state (context) can be **saved** when the application is about to be suspended
   - Execution state (context) can be **restored** when the application is to continue
```rust
1//os/src/task/context.rs
2 pub struct TaskContext {
3 ra: usize, //function return address
4 sp: usize, //user stack pointer
5 s: [usize; 12], //belongs to the register set s0~s11 saved by the Callee function
6}
```


---

#### Task context data structure

```rust
1//os/src/task/context.rs
2 pub struct TaskContext {
3 ra: usize,
4 sp: usize,
5 s: [usize; 12],
6}
```
```rust
// os/src/trap/context.rs
pub struct TrapContext {
     pub x: [usize; 32],
     pub sstatus: Sstatus,
     pub sepc: usize,
}
```

![bg right:60% 100%](figs/more-task-multiprog-os-detail.png)

---

#### Different types of context
   - Function call context
   - Trap context
   - Task context

![bg right:60% 90%](figs/contexts-on-stacks.png)

---

#### Task (Task) context vs system call (Trap) context

A task switch is a switch between Trap control flows in the kernel from two different applications
- Task switching does not involve **privilege level** switching; Trap switching involves privilege level switching;
- Task switching only saves **partial registers** that the callee function agreed by the compiler should save; while Trap switching needs to save all general-purpose registers;
- Both task switching and trap switching are **transparent to the application**

---
#### Control Flow
- Program control flow (Flow of Control or Control Flow) -- Compilation principle
     - The **execution sequence** in units of instructions, statements or basic blocks of a program.
- Processor control flow -- Principles of computer composition
     - The **control transfer sequence** of the program counter in the processor.
---
#### Common Control Flow: Control Flow from an Application Programmer's Perspective

- Control flow is the **sequence of execution** of an application program written by the application programmer, and these sequences are preset by the programmer.
- Called **Common Control Flow** (CCF, Common Control Flow)

---
#### Abnormal Control Flow: Control Flow from an Operating System Programmer's Perspective

- During the execution of the application program, if a system call request is issued, or a peripheral interrupt, CPU exception, etc. occurs, the previous instruction is still in the code segment of the application program, and the latter instruction will run to the code segment of the operating system gone.
- This is a "**mutation**" of the control flow, that is, the control flow breaks away from the execution environment where it is located, and produces a **switch of the execution environment**.
- This "mutant" control flow is called **Exceptional Control Flow** (ECF, Exceptional Control Flow).


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Control flow context (the state of the execution environment)

From the perspective of hardware, the execution process of ordinary control flow or abnormal control flow
* Starting from the execution of an instruction at the beginning of the control flow, the contents of all physical resources accessible by the instruction, including all general-purpose registers, privilege-related special registers, and memory accessed by the instruction, will follow the execution of the instruction And change gradually.

- The physical resource content of the control flow after executing a certain instruction, that is, the physical/virtual resource content that ensures that the control flow instruction can continue to be executed correctly at the next moment is called ***Control Flow Context (Context)***, it can also be Called the state of the execution environment in which the flow of control occurs.

For the currently practiced OS, there is no virtual resource, and the content of the physical resource is *** general register/CSR register ***

---
<style scoped>
{
  font-size: 33px
}
</style>

#### Control flow context (the state of the execution environment)

- function call context
     - Control flow context during a function call (executing a function switch)
- interrupt/exception/trapped in context
     - The context of control flow when handling interrupt/exception/trapped switching code in the operating system
- task (process) context
     - The context of control flow when tasks (processes) in the operating system execute related switching codes

---
**Outline**

...

4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
* Task switching
### Trap control flow switching
* Collaborative Scheduling
6. Coelophysis OS: time-sharing multitasking OS

---
<style scoped>
{
  font-size: 30px
}
</style>

#### The OS Challenge: Task Switching
Perform hacker-level operations between two Trap control flows belonging to different tasks, that is, perform **Trap context switching** to achieve task switching.

- Where is the Trap context?
- Where is the task context?
- How to switch tasks?
- Where should task switching occur?
- Can I switch back after switching tasks?

![bg right:45% 95%](figs/task-context.png)

---
#### Trap control flow switching: pause operation
- a special function `__switch()`
- During the period after calling `__switch()` until it returns, the original Trap control flow A will be suspended and switched out, and the CPU will run another Trap control flow B that is applied in the kernel.
![bg right 95%](figs/task-context.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Trap control flow switching: resume operation
- A special function `__switch()`
- Then at an appropriate time, the original Trap control flow A will switch back from a certain Trap control flow C (probably not the B it switched to before) to continue execution and finally return.

From an implementation point of view, the core difference between a `__switch()` function and a normal function is simply that it **switches** the stack.
![bg right:45% 95%](figs/task-context.png)


---
#### Trap control flow switching function `__switch()`
![w:800](figs/switch.png)


---
#### Trap control flow switching process: state before switching
Stage [1]: Before the Trap control flow A calls `__switch()`, A’s **kernel stack** only has the Trap context and the call stack information of the Trap processing function, and B was switched out before;
![bg right:55% 100%](figs/switch.png)


---
#### Trap control flow switching process: save A task context
Stage [2]: A saves the **CPU current register snapshot** in the A task context space;
![bg right:55% 100%](figs/switch.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Trap control flow switching process: restore B task context

Phase [3]: Read the B task context pointed to by next_task_cx_ptr, restore the ra register, s0~s11 register and sp register.
* After this step is completed, `__switch()` can execute a function across two control flows, that is, the switching of the control flow is realized by **switching the stack**.
![bg right:50% 100%](figs/switch.png)


---
#### Trap control flow switching process: execute B task code
Stage [4]: After the CPU executes the ret assembly pseudo-instruction and completes the return of the `__switch()` function, task B can continue to execute downwards from the position where `__switch()` was called.
* `__switch()` is switched to the kernel stack of task B by restoring the sp register, realizing the switching of control flow, so that a function can be executed across two control flows.
![bg right:53% 100%](figs/switch.png)



---
#### Interface of `__switch()`
```
  1 // os/src/task/switch.rs
  2 
  3 global_asm!(include_str!("switch.S"));
  4
  5 use super::TaskContext;
  6
  7 extern "C" {
  8 pub fn __switch(
  9 current_task_cx_ptr: *mut TaskContext,
10 next_task_cx_ptr: *const TaskContext
11);
12}
```


---
#### Implementation of `__switch()`

```
12 __switch:
13 # stage [1]
14 # __switch(
15 # current_task_cx_ptr: *mut TaskContext,
16 # next_task_cx_ptr: *const TaskContext
17#)
18 # stage [2]
19 # save kernel stack of current task
20 sd sp, 8(a0)
21 # save ra & s0~s11 of current execution
22 sd ra, 0(a0)
23.set n, 0
24 .rept 12
25 SAVE_SN %n
26. set n, n + 1
27.endr

```


---
#### Implementation of `__switch()`

```
28 # stage [3]
29 # restore ra & s0~s11 of next execution
30 ld ra, 0(a1)
31. set n, 0
32 .rept 12
33 LOAD_SN %n
34.set n, n + 1
35.endr
36 # restore kernel stack of next task
37 ld sp, 8(a1)
38 # stage [4]
39 ret
```



---
**Outline**

...

4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
* Task switching
* Trap control flow switching
### Cooperative Scheduling
6. Coelophysis OS: time-sharing multitasking OS

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Task Control Block
The operating system manages the collection of information used to control the running of processes
```Rust
pub struct TaskControlBlock {
     pub task_status: TaskStatus,
     pub task_cx: TaskContext,
}
```
- Task management module
```Rust
struct TaskManagerInner {
     tasks: [TaskControlBlock; MAX_APP_NUM],
     current_task: usize,
}
```
![bg right:50% 100%](figs/more-task-multiprog-os-detail.png)


---
#### Cooperative Scheduling
- `sys_yield` and `sys_exit` system calls
```rust
pub fn sys_yield() -> isize {
     suspend_current_and_run_next();
     0
}
pub fn sys_exit(exit_code: i32) -> ! {
     println!("[kernel] Application exited with code {}", exit_code);
     exit_current_and_run_next();
     panic!("Unreachable in sys_exit!");
}
```

---
#### Cooperative Scheduling

- `sys_yield` and `sys_exit` system calls

```rust
// os/src/task/mod.rs

pub fn suspend_current_and_run_next() {
     mark_current_suspended();
     run_next_task();
}

pub fn exit_current_and_run_next() {
     mark_current_exited();
     run_next_task();
}
```


---
#### Cooperative Scheduling

- `sys_yield` and `sys_exit` system calls
```Rust
  fn run_next_task(&self) {
      …
     unsafe {
         __switch(
             current_task_cx_ptr, //current task context
             next_task_cx_ptr, //Next task context
         );
     }
```

---
#### Enter user mode for the first time
**Q: How to achieve it?**

If it can be done, we will realize the initial dragon operating system that supports multi-program cooperative scheduling

---
**Outline**

1. Experimental objectives and steps
2. Multi-channel batch processing operating system design
3. Application Design
4. Sawtooth OS: Support application loading
5. Shichulong OS: Support multi-program cooperative scheduling
### 6. Coelophysis OS: time-sharing multitasking OS

---

#### Coelophysis OS: time-sharing multitasking OS

The Triassic "Coelophysis" operating system – Timesharing OS can completely preempt the execution of applications, so that multiple applications can be executed in a fair and efficient time-sharing manner, and the overall efficiency of the system can be improved.

![bg right:50% 100%](figs/time-task-multiprog-os-detail.png)

---

#### The basic idea of time-sharing multitasking operating system
- Set clock interrupt
- Count the time slice used by the task after receiving the clock interrupt
- After the time slice is used up, switch tasks
![bg right:50% 100%](figs/time-task-multiprog-os-detail.png)
---
#### Clock Interrupts and Timers
- set clock interrupt
```rust
// os/src/sbi.rs
pub fn set_timer(timer: usize) {
      sbi_call(SBI_SET_TIMER, timer, 0, 0);
  }
// os/src/timer.rs
pub fn set_next_trigger() {
     set_timer(get_time() + CLOCK_FREQ / TICKS_PER_SEC);
}
pub fn rust_main() -> ! {
     trap::enable_timer_interrupt();
     timer::set_next_trigger();
}
```


---
#### Preemptive Scheduling

```rust
// os/src/trap/mod.rs trap_handler function
 …
match cause. cause() {
     Trap::Interrupt(Interrupt::SupervisorTimer) => {
         set_next_trigger();
         suspend_current_and_run_next();
     }
}
```

In this way, we have realized the time-sharing and multi-tasking Coelophysis operating system

---
### Summary
- Multiprogramming & time sharing multitasking
- Cooperative Scheduling & Preemptive Scheduling
- Task and task switching
- Interrupt mechanism

---
<style scoped>
{
  font-size: 30px
}
</style>

### Course Experiment 1

* Create an experimental submission repository
     * Tsinghua git access entrance: [UniLab Platform](https://lab.cs.tsinghua.edu.cn/unilab/home)
     * [Access Entrance] (https://www.yuque.com/xyong-9fuoz/qczol5/opl4y4#DiUQ0) of XuetangX students (to be added)
         * rCore, uCore-RV, uCore-x86
* Experimental tasks
     * Chapter 3: Multiprogramming and time-sharing multitasking -> chapter3 practice -> get task information -> add a system call `sys_task_info()`
* Experiment submission requirements
     * The 11th day after the task is assigned (October 16, 2022);