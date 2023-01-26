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
### Section 4 Practice: Operating System Supporting Processes

Process OS (POS)

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---

**Outline**

### 1. Experimental objectives and steps
- Experiment objectives
- Practical steps
2. Code structure
3. Application Design
4. Kernel programming

![bg right:57% 100%](figs/process-os-detail.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Previous Goals

Improve performance, simplify development, strengthen security
- Address Space OS
     - APP does not need to consider the initial execution address at runtime, and isolates the address space accessed by APP
- Multiprog & time-sharing
     - Let the APP effectively share the CPU to improve the overall performance and efficiency of the system
- BatchOS: isolate APP from OS, strengthen system security, and improve execution efficiency
- LibOS: Isolate APP from HW, simplifying the difficulty and complexity of applications accessing hardware

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Experiment Objectives

Enhanced process management and resource management, improved performance, simplified development, enhanced security

- Integrate the previous privilege level, address space, and tasks to form a process
- The process becomes the owner of the resource
- Extended process dynamic features, which can issue the following system call requests at the application level:
    - Dynamically create child processes
    - Dynamically construct new processes
    - child process exits / parent process waits for child process to exit

---

#### Experiment Requirements

- Understand process concepts
- Understand the design and implementation of the dynamic management mechanism of the process
- Preliminary understanding of process scheduling
- Master the writing and use of shell applications
- Can write an operating system that supports processes

<!-- The most intelligent Cretaceous "Troodon" operating system troodon -->

![bg right:30% 100%](figs/troodon.png)



---

#### General idea

![bg right:70% 90%](figs/process-os-detail.png)


---

#### General idea

![bg right:70% 85%](figs/process-os-key-structures.png)

---

#### General idea

- Compilation: The application program and the kernel are compiled independently and merged into one image
- Compilation: Different applications can use a unified starting address
- Structure: system call service, **process management and initialization**
- Construction: Establish a virtual memory space based on the page table mechanism
- Run: privilege level switching, process and OS switching
- Run: switch address spaces, access data across address spaces


---
<style scoped>
{
  font-size: 30px
}
</style>

#### History background
- 1965: Describes the future MULTICS operating system
   - Led by Prof. Fernando J. Corbató at MIT
   - Participating units: MIT, GE (General Electric Company), AT&T Bell Labs
   - Proposed **process dynamic management idea**, inspired and created UNIX
- 1971: Thompson shell
   - The first **UNIX Shell** written by Ken Thompson
   - Designed according to minimalism, the syntax is very simple, it is a simple command line interpreter
   - Many of its features influenced the development of command-line interfaces for later operating systems
---

#### Practical steps
```
git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
cd rCore-Tutorial-v3
git checkout ch5
cd os
make run
```

---
#### Practical steps
```
[RustSBI output]
...
yield
***************/
Rust user shell
>>
```
After the operating system starts the ``shell``, the user can execute the application by typing the application name in the ``shell``.


---

#### Software Architecture

- Manage processes
     - create
     - Recycle
     - fork
     - exec
  
![bg right:55% 100%](figs/process-os-detail.png)

---

**Outline**

1. Experimental objectives and steps
### 2. Code structure
3. Application Design
4. Kernel programming

![bg right:55% 100%](figs/process-os-detail.png)

---

#### Improve OS
```
├── os
     ├── build.rs (modification: application builder based on application name)
     └── src
     ├── loader.rs (modification: application loader based on application name)
   ├── main.rs (modified)
          ├── mm (Modification: In order to support the system calls in this chapter, some enhancements are made to this module)
```

---

#### Improve OS
```
├── os
     └── src
   ├── syscall
   ├──fs.rs (modification: add sys_read)
              ├── mod.rs (modification: new system call distribution processing)
              └── process.rs (modified: add sys_getpid/fork/exec/waitpid)
   ├── task
              ├── manager.rs (new: task manager, which is part of the task manager function in the previous chapter)
   ├── mod.rs (modification: adjust the original interface implementation to support process)
              ├── pid.rs (New: Rust abstraction of process identifiers and kernel stack)
              ├── processor.rs (new: processor management structure ``Processor``, which is part of the task manager function in the previous chapter)
             └── task.rs (modification: task control block supporting process management mechanism)
          └── trap
               ├── mod.rs (modification: modify the implementation of system calls to support process system calls)
```


---

**Outline**

1. Experimental objectives and steps
2. Code structure
### 3. Application Design
4. Kernel programming

![bg right:55% 90%](figs/process-os-detail.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Understanding Processes

- Application Angle
     - **Process** is the running application
  
- OS Angle
     - **Process** is an execution process applied on its address space
        - A process owns resources, and the operating system manages its resources according to the execution state of the process
![bg right:45% 100%](figs/seg-addr-space.png)

---

#### Process management system calls

```
/// Function: The current process forks a child process.
/// Return value: return 0 for the child process, and return the PID of the child process for the current process
/// syscall ID: 220
pub fn sys_fork() -> isize;
```
```
/// Function: Clear the address space of the current process and load a specific executable file, and start its execution after returning to the user state.
/// Parameter: path gives the name of the executable file to be loaded;
/// Return value: If there is an error (such as no executable file with the same name), return -1, otherwise it should not return.
/// syscall ID: 221
pub fn sys_exec(path: &str) -> isize;
```
<!-- ![bg right:50% 100%](figs/app-as-full.png) -->



---

#### Process management system calls

```
/// Function: The current process waits for a child process to become a zombie process, reclaims all its resources and collects its return value.
/// Parameter: pid indicates the process ID of the child process to wait for, if it is -1, it means waiting for any child process;
/// exit_code indicates the address of saving the return value of the child process, if this address is 0, it means that it does not need to be saved.
/// Return value: If the child process to be waited for does not exist, return -1; otherwise, if the child process to be waited for has not ended, return -2;
/// Otherwise returns the process ID of the terminated child process.
/// syscall ID: 260
pub fn sys_waitpid(pid: isize, exit_code: *mut i32) -> isize;
```


---

#### Application ``shell`` execution flow
1. Get the string (namely the file name) through ``sys_read``
2. Create a child process through ``sys_fork``
3. Create a new application process through ``sys_exec`` in the child process
4. Wait for the end of the child process through ``sys_waitpid`` in the parent process
5. Jump to the first step for loop execution

---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental objectives and steps
2. Code structure
3. Application Design
### 4. Kernel programming
- Application linking and loading support
- Core data structures
- Implementation of process management mechanism

![bg right:55% 90%](figs/process-os-detail.png)

---

#### Application linking and loading support

During the process of compiling the operating system, the following link_app.S file will be generated
```
  3_num_app:
  4 .quad 15 #Number of applications
  7  …
  9 _app_names: #app0 name
10.string "exit"
12  …
17 app_0_start: #app0 start position
18.incbin "../user/target/riscv64gc-unknown-none-elf/release/exit"
19 app_0_end: End position of #app0
```


---

#### Application loading based on application name

In the loader loader.rs, analyze the content in link_app.S, and use a globally visible **read-only** vector ``APP_NAMES`` to store the names of all applications in memory in order, for the purpose of passing through the exec system The call to create a new process is done in advance.


---
<style scoped>
{
  font-size: 30px
}
</style>
**Outline**

1. Experimental objectives and steps
2. Code structure
3. Application Design
4. Kernel programming
- Application linking and loading support
### Core Data Structures
- Implementation of process management mechanism

![bg right:55% 100%](figs/process-os-detail.png)

---

#### Relationship between core data structures

![bg right:60% 100%](figs/process-os-key-structures.png)

---

#### Process Control Block TCB

The corresponding implementation of the process abstraction is the process control block -- TCB ``TaskControlBlock``
```rust
pub struct TaskControlBlock {
     // immutable
     pub pid: PidHandle, // process id
     pub kernel_stack: KernelStack, // process kernel stack
     //mutable
     inner: UPSafeCell<TaskControlBlockInner>,//process internal management information
}
```



---

#### Process Control Block TCB

The corresponding implementation of the process abstraction is the process control block -- TCB ``TaskControlBlock``
```rust
pub struct TaskControlBlockInner {
     pub trap_cx_ppn: PhysPageNum, // The physical page number of the context page trapped
     pub base_size: usize, // the user stack top of the process
     pub task_cx: TaskContext, // process context
     pub task_status: TaskStatus, // process execution status
     pub memory_set: MemorySet, // process address space
     pub parent: Option<Weak<TaskControlBlock>>, // parent process control block
     pub children: Vec<Arc<TaskControlBlock>>, // child process task control block group
     pub exit_code: i32, // exit code
}
```

---

#### Process Manager ``TaskManager``

- The task manager itself is only responsible for managing all ready processes

```rust
pub struct TaskManager {
     ready_queue: VecDeque<Arc<TaskControlBlock>>, // linked list of ready state task control blocks
}
```

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Processor Management Structure

Processor management structure ``Processor`` describes the CPU execution state
```rust
pub struct Processor {
     current: Option<Arc<TaskControlBlock>>, // the task being executed on the current processor
     idle_task_cx: TaskContext, // idle task
}
```
- Responsibility for maintaining the CPU state is split from the task manager `TaskManager`
- Maintain tasks running on a processor, view its information or replace it
- `Processor` has an idle control flow, the function is to try to select a task from the task manager to execute on the current CPU core, and has its own CPU to start the kernel stack



---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental objectives and steps
2. Code structure
3. Application Design
4. Kernel programming
- Application linking and loading support
- Core data structures
### Implementation of process management mechanism

![bg right:50% 100%](figs/process-os-detail.png)

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Process management mechanism implementation overview

1. Create an initial process: create the first user mode process `initproc`
2. Process generation mechanism: introduce process-related system calls `sys_fork`/`sys_exec`
3. Process scheduling mechanism: process active/passive switching
4. Process resource recycling mechanism: Call `sys_exit` to exit or save the exit code after the process terminates
5. Process resource recycling mechanism: the parent process collects the information of the process through `sys_waitpid` and reclaims its resources
6. Character input mechanism: get character input through `sys_read` system call


---

#### Create the initial process

```rust
lazy_static! {
     pub static ref INITPROC: Arc<TaskControlBlock> = Arc::new(
         TaskControlBlock::new(get_app_data_by_name("initproc").unwrap()));
}
pub fn add_initproc() {
     add_task(INITPROC. clone());
}
```
- `TaskControlBlock::new` will parse the ELF execution file format of `initproc`, and establish the application address space, kernel stack, etc. to form a ready process control block
- `add_task` will add the process control block to the ready queue



---
<style scoped>
{
  font-size: 30px
}
</style>

#### Create a new process `fork()`

Copy the contents of the parent process and construct a new process control block

```rust
pub fn fork(self: &Arc<TaskControlBlock>) -> Arc<TaskControlBlock> {...}
```
- Create a new page table and copy the contents of the parent process address space
- create new trapping context
- Create a new application kernel stack
- Create task context
- Establish parent-child relationship
- set `0` as `fork` return code

---

#### Load new application `exec()`

Replaces the contents of the original application address space with code and data from the new application's ELF executable

```rust
pub fn exec(&self, elf_data: &[u8]) {...}
```
- Reclaim the existing application address space, and directly replace the existing application address space with a new address space based on the ELF file
- Modify the Trap context of the process control block, and initialize the parsed application entry point, user stack location and some kernel information


---
<style scoped>
{
  font-size: 32px
}
</style>

#### Process Scheduling Mechanism

Pause the current task and switch to the next one
- Timing
    - When the `sys_yield` system call
    - when the time slice of the process runs out
- Operate
    - Execute `suspend_current_and_run_next` function
       - Take out the currently executing task, modify its status, and put it at the end of the ready queue
       - Then call the schedule function to trigger scheduling and switch tasks

---

#### Process resource recovery mechanism

Process exits `exit_current_and_run_next`
- The current process control block is taken out from ``PROCESSOR``, modifying it as a zombie process
- The exit code `exit_code` is written in the process control block
- Hang all child processes into the set of child processes in `initproc`
- Release application address space
- Then call the schedule function to trigger scheduling and switch tasks


---
<style scoped>
{
  font-size: 32px
}
</style>

#### Process resource recovery mechanism

Waiting for child process to exit `sys_waitpid`

- returns -1 when there is no child process with process ID pid (pid==-1 or > 0)
- When there is a zombie child process whose process ID is pid, the child process is recycled normally, the child process pid is returned, and the exit code parameter is updated to exit_code
- When the child process has not exited, it returns -2. After the user library sees that it is -2, it further calls the sys_yield system call to make the parent process enter the waiting state
- Before returning, release the process control block of the child process

---

#### Character Input Mechanism

```rust
pub fn sys_read(fd: usize, buf: *const u8, len: usize) -> isize {
    c=sbi::console_getchar(); ...}
  
```
- Currently only supports reading one character at a time
- Call the interface `console_getchar` provided by the sbi submodule to get input from the keyboard

---
#### Operating system POS that supports processes
- The relationship between process concept and process realization
- Process management mechanism
- Basic scheduling mechanism
- Can write Troodon OS
![bg right:30% 100%](figs/troodon.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

### Summary

**Outline**

1. Experimental objectives and steps
2. Code structure
3. Application Design
4. Kernel programming
- Application linking and loading support
- Core data structures
- Implementation of process management mechanism

![bg right:45% 100%](figs/process-os-detail.png)