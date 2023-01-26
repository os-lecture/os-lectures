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

# Lecture 10 Inter-process communication

## Section 2 OS that supports IPC
IPC OS (IOS)

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

Fall 2022

---
<style scoped>
{
  font-size: 28px
}
</style>

**Outline**

### 1. Experimental arrangement
- Experiment objectives
- General idea
- History background
- Practical steps
2. Code structure
3. Pipeline design and implementation
4. Signal design and implementation

![bg right:50% 95%](figs/ipc-os-detail-2.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Past experiment goals

Improve performance, simplify development, enhance security, support data persistence
- Filesystem OS: support for persistent data storage
- Process OS: Enhanced process management and resource management
- Address Space OS: Isolate the memory address space accessed by APP
- Multiprog & time-sharing OS: Let APP share CPU resources
- BatchOS: isolate APP from OS, strengthen system security, and improve execution efficiency
- LibOS: Isolate APP from HW, simplifying the difficulty and complexity of applications accessing hardware

---
<style scoped>
{
  font-size: 28px
}
</style>

#### Experiment Objectives
Support application flexibility and inter-process interaction
- Extended file abstraction: Pipe, Stdout, Stdin
- Interprocess data exchange in the form of files
- Serial port input and output in the form of files
- Signal implements inter-process asynchronous notification mechanism
- Number of system calls: 11 --> 17
   - Pipeline: 2, used to transfer data
   - Signals: 4, for notification

![bg right:35% 95%](figs/ipc-os-detail-2.png)

---

#### Experiment Requirements

- Understand the file abstraction
- Understand the design and implementation of IPC mechanism
   - pipe
   - signal
- Can write OS that supports IPC

<!-- Velociraptor possesses stealth and cunning. Velociraptor is smarter, has teamwork ability, and is good at teamwork Operating system -->

![bg right:40% 80%](figs/velociraptor.png)



---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental arrangement
- Experiment objectives
### General idea
- History background
- Practical steps
2. Code structure
3. Pipeline design and implementation
4. Signal design and implementation

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---

#### Issues that need to be considered in pipeline implementation

- What is a pipe?
- How to access the pipeline?
- How are pipelines managed?

![w:1100](figs/pipe-fds.png)

---
<style scoped>
{
  font-size: 28px
}
</style>

#### Understanding Pipelines

**A pipe is a block of memory in the kernel**
- Sequential writing/reading of byte streams

**Pipelines can be abstracted as files**
- the process contains the pipe file descriptor
   - The interface of the `File` Trait for pipes
   - read/write
- The system call for the application to create the pipeline
   - `sys_pipe`

![bg right:40% 100%](figs/pipe-fds-close.png)


---

#### Pipeline sample program (user mode)
```rust
...//usr/src/bin/pipetest.rs
static STR: &str = "Hello, world!" //String global variable
pub fn main() -> i32 {
     let mut pipe_fd = [0usize; 2]; // fd array containing two elements
     pipe(&mut pipe_fd); // create pipe
     if fork() == 0 { // child process, read from parent
         close(pipe_fd[1]); // close write_end
         let mut buffer = [0u8; 32]; //A byte array containing 32 bytes
         let len_read = read(pipe_fd[0], &mut buffer) as usize; //read pipe
     } else { // parent process, write to child
         close(pipe_fd[0]); // close read end
         write(pipe_fd[1], STR.as_bytes()); //write pipe
         let mut child_exit_code: i32 = 0;
         wait(&mut child_exit_code); //The parent process waits for the child process to end
     }
...
```
---


#### The relationship between pipelines and processes
- `pipe` is one of the resources of the process control block

![bg right:60% 95%](figs/process-os-key-structures-file-ipc.png)

---

#### Issues that need to be considered in signal implementation
- what is the signal?
- How to use the signal?
- How to manage signals?

![bg right:60% 90%](figs/signal.png)

<!-- Those things about linux signal http://blog.chinaunix.net/uid-24774106-id-4061386.html -->

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Understanding Signals
`signal` is the software interrupt that the kernel notifies the application

**Preparation Phase**
- Set the integer number value of `signal`
- Create a routine `signal_handler` that responds to a `signal` number value

**Execution phase**
- Send a signal to a process, interrupt the current execution of the process, and go to `signal_handler` for execution

![bg right:40% 100%](figs/signal.png)



---

#### Signal sample program (user mode)
```rust
...//usr/src/bin/sig_simple.rs
fn func() { //signal_handler
     println!("user_sig_test succsess");
     sigreturn(); // Return to the position before the signal processing to continue execution
}
pub fn main() -> i32 {
     let mut new = SignalAction::default(); //new signal configuration
     let old = SignalAction::default(); //Old signal configuration
     new.handler = func as usize; //Set up a new signal handling routine
     if sigaction(SIGUSR1, &new, &old) < 0 { //setup signal_handler
         panic!("Sigaction failed!");
     }
     if kill(getpid() as usize, SIGUSR1) <0{ //send SIGUSR1 to itself
       ...
     }
...
```

---

#### The relationship between signals and processes
- `signal` is one of the resources of the process control block

![bg right:60% 90%](figs/process-os-key-structures-file-ipc.png)



---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental arrangement
- Experiment objectives
- General idea
### History background
- Practical steps
2. Code structure
3. Pipeline design and implementation
4. Signal design and implementation

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---
<style scoped>
{
  font-size: 30px
}
</style>

#### Pipes: The Most Remarkable Invention in Unix

- The concept of pipes comes from Douglas McIlroy at Bell Laboratories, who in an internal document written in 1964 proposed the idea of chaining and screwing together multiple programs "like garden hoses" so that data could flow in Flow in different programs.
- Around the second half of 1972, after listening to Douglas McIlroy's nagging about pipelines, Ken Thompson had an idea and quickly implemented the pipeline mechanism in UNIX.
 
![bg right:35% 90%](figs/douglas-mcllroy.jpg)

<!-- Douglas McIlroy (born 1932) is a mathematician, engineer and programmer. Since 2007 he is an adjunct professor of computer science at MIT and received his Ph.D.
McIlroy is best known for the original Unix pipeline implementation, software component parts and several Unix tools such as computer virus, diff, sort, join, talk, and tr. -->


---
#### Signal: Error-Prone Software Interrupts in Unix

Signals have been around since the very first versions of Unix, just a little differently than what we know today, requiring different system calls to catch different types of signals. After version 4, improved to catch all signals with one system call.

![bg right:35% 90%](figs/douglas-mcllroy.jpg)

<!-- https://venam.nixers.net/blog/unix/2016/10/21/unix-signals.html
https://unix.org/what_is_unix/history_timeline.html
https://en.wikipedia.org/wiki/Signal_(IPC) -->

---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental arrangement
- Experiment objectives
- general idea
- History background
### Practical steps
2. Code structure
3. Pipeline design and implementation
4. Signal design and implementation

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---

#### Practical steps
```
git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
cd rCore-Tutorial-v3
git checkout ch7
cd os
make run
```


---
#### Reference output
```
[RustSBI output]
...
filetest_simple
fantastic_text
***************/
Rust user shell
>>
```
After the operating system starts the ``shell``, the user can execute the application by typing the application name in the ``shell``.

---
#### Test case pipetest
Here we run the test case pipetest in this chapter:
```
>> pipe test
Read OK, child process exited!
pipe test passed!
>>
```
The parent and child processes of this application complete the transmission of the string `"Hello, world!"` through the pipe.

---
#### Test case sig_simple

Here we run the test case sig_simple of this chapter:

```
>> sig_simple
signal_simple: sigaction
signal_simple: kill
user_sig_test succsess
signal_simple: Done
>>
```
This application establishes a signal processing routine `func` for the `SIGUSR1` signal, and then sends a signal `SIGUSR1` to itself through `kill`, and finally `func` will be called.


---

**Outline**

1. Experimental arrangement
### 2. Code structure
3. Pipeline design and implementation
4. Signal design and implementation

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---

#### User Code Structure
```
└── user
     └── src
         ├── bin
         │ ├── pipe_large_test.rs (new: large data pipeline transmission)
         │ ├── pipetest.rs (new: parent-child process pipeline transmission)
         │ ├── run_pipe_test.rs (new: pipeline test)
         │ ├── sig_tests.rs (new: multi-directional test signal mechanism)
         │ ├── sig_simple.rs (new: send a signal to yourself)
         │ ├── sig_simple2.rs (new: parent process sends signal to child process)
         ├── lib.rs (two new system calls: sys_close/sys_pipe/sys_sigaction/sys_kill...)
         └── syscall.rs (two new system calls: sys_close/sys_pipe/sys_sigaction/sys_kill...)
```

---
#### Kernel code structure
```
├── fs (new: file system submodule fs)
│ ├── mod.rs (Abstract File Trait that contains files that have been opened and can be read and written by the process)
│ ├── pipe.rs (implements the first branch of File Trait - a pipe that can be used for inter-process communication)
│ └── stdio.rs (implements the second branch of File Trait - standard input/output)
├── mm
│ └── page_table.rs (New: Application address space buffer abstraction UserBuffer and its iterator implementation)
├── syscall
│ ├── fs.rs (modification: adjust the implementation of sys_read/write, add sys_close/pipe)
│ ├── mod.rs (modification: adjust syscall distribution)
├── task
│ ├── action.rs (definition and default behavior of signal processing SignalAction)
│ ├── mod.rs (signal processing related functions)
│ ├── signal.rs (signal value definition for signal processing, etc.)
│ └── task.rs (modification: add signal-related content in the task control block)
└── trap
├── mod.rs (signal processing when entering/exiting the kernel)
```


---

**Outline**

1. Experimental arrangement
2. Code structure
### 3. Pipeline design and implementation
4. Signal design and implementation

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---
<style scoped>
{
  font-size: 32px
}
</style>

#### Pipeline design and implementation
Based on file abstraction, supports I/O redirection
1. [K] Implement file-based standard input/output
2. [K] Implement file-based implementation pipeline
3. [U] Support command line parameters
4. [U] Support "|" symbol

![bg right:40% 100%](figs/tcb-ipc-standard-file.png)

---
#### Standard Documentation

1. Implement file-based standard input/output
  - FD: 0 -- Stdin ; 1/2 -- Stdout
  - Implement the File interface
    - read -> call(SBI_CONSOLE_GETCHAR)
    - write -> call(SBI_CONSOLE_PUTCHAR)


![bg right:40% 100%](figs/tcb-ipc-standard-file.png)






---
#### Standard file initialization

2. Initialize `fd_table` when creating TCB

```rust
TaskControlBlock::fork(...)->... {
   ...
   let task_control_block = Self {
       ...
           fd_table: vec![
               // 0 -> stdin
               Some(Arc::new(Stdin)),
               // 1 -> stdout
               Some(Arc::new(Stdout)),
               // 2 -> stderr
               Some(Arc::new(Stdout)),
           ],
...
```

![bg right:35% 100%](figs/tcb-ipc-standard-file.png)

---
#### Standard file creation in the fork implementation

3. Copy `fd_table` when `fork`

```rust
TaskControlBlock::new(elf_data: &[u8]) -> Self{
   ...
     // copy fd table
     let mut new_fd_table = Vec::new();
     for fd in parent_inner.fd_table.iter() {
         if let Some(file) = fd {
             new_fd_table.push(Some(file.clone()));
         } else {
             new_fd_table. push(None);
         }
     }
```

![bg right:35% 100%](figs/tcb-ipc-standard-file.png)



---
#### Pipeline file

1. Pipeline system calls

```rust
/// Function: Open a pipe for the current process.
/// Parameter: pipe indicates the address space of the application
/// of a usize array of length 2
/// The starting address, the kernel needs to read the end of the pipeline in order
/// Write the file descriptor of the write end into the array.
/// Return value: -1 if an error occurs,
/// Otherwise return 0.
/// The possible cause of the error is: the incoming address is invalid.
/// syscall ID: 59
pub fn sys_pipe(pipe: *mut usize) -> isize;
```

![bg right:35% 100%](figs/tcb-ipc-standard-file.png)

---
#### Pipeline file

2. Create the Buffer in the pipeline

```rust
pub struct PipeRingBuffer {
     arr: [u8; RING_BUFFER_SIZE],
     head: usize,
     tail: usize,
     status: RingBufferStatus,
     write_end: Option<Weak<Pipe>>,
}

make_pipe() -> (Arc<Pipe>, Arc<Pipe>) {
     let buffer = PipeRingBuffer::new();
     let read_end = Pipe::read_end_with_buffer();
     let write_end = Pipe::write_end_with_buffer();
     ...
     (read_end, write_end)
```
![bg right:35% 100%](figs/tcb-ipc-standard-file.png)



---
#### Pipeline file

3. Implement file-based input/output
  - Implement the File interface
```rust
     fn read(&self, buf: UserBuffer) -> usize {
        *byte_ref = ring_buffer. read_byte();
     }
     fn write(&self, buf: UserBuffer) -> usize {
       ring_buffer.write_byte( *byte_ref );
     }
```

![bg right:35% 100%](figs/tcb-ipc-standard-file.png)


---
#### Command line parameters for the exec system call
- System call interface for sys_exec needs to change
```rust
// Added the args parameter
pub fn sys_exec(path: &str, args: &[*const u8]) -> isize;
```
- The command line parameter split of the shell program
```rust
// Get parameters from a line of string
let args: Vec<_> = line.as_str().split(' ').collect();
// Execute the sys_exec system call with the application name and parameter address
exec(args_copy[0].as_str(), args_addr.as_slice())
```
![bg right:35% 100%](figs/tcb-ipc-standard-file.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Command line parameters for the exec system call

- Push the obtained parameter string onto the user stack
```rust
impl TaskControlBlock {
  pub fn exec(&self, elf_data: &[u8], args: Vec<String>) {
    ...
    // push arguments on user stack
  }
```
- The a0/a1 registers in the Trap context, let a0 indicate the number of command line parameters, and a1 indicate the starting address of argv_base in the figure, which is the blue area.

![bg right:30% 100%](figs/user-stack-cmdargs.png)



---

#### Command line parameters for the exec system call

```rust
pub extern "C" fn _start(argc: usize, argv: usize) -> ! {
    //Get the command line number of the application argc, get the command line parameters of the application into v
    // Execute the main function of the application
    exit(main(argc, v. as_slice()));
}
```
![bg right:30% 100%](figs/user-stack-cmdargs.png)



---
#### Redirection
- Copy file descriptor system call
```rust
/// Function: copy an already opened file in the process
/// A copy is allocated to a new file descriptor.
/// Parameter: fd represents the file descriptor of an open file in the process.
/// Return value: If there is an error, return -1, otherwise you can access the opened
/// Open a new file descriptor for the file.
/// The possible cause of the error is: the incoming fd does not correspond to a valid
/// Open the file.
/// syscall ID: 24
pub fn sys_dup(fd: usize) -> isize;
```
<!-- ![bg right:30% 100%](figs/user-stack-cmdargs.png) -->


---
#### Redirection
- Copy file descriptor system call
```rust
pub fn sys_dup(fd: usize) -> isize {
   ...
   let new_fd = inner.alloc_fd();
   inner.fd_table[new_fd] = inner.fd_table[fd];
   newfd
}
```
<!-- ![bg right:30% 100%](figs/user-stack-cmdargs.png) -->


---
#### Shell redirection "$A | B"
```rust
// user/src/bin/user_shell.rs
{
   let pid = fork();
     if pid == 0 {
         let input_fd = open(input, ...); // input redirection -- B child process
         close(0); //close file descriptor 0
         dup(input_fd); //File descriptor 0 and file descriptor input_fd point to the same file
         close(input_fd); //Close the file descriptor input_fd
         //or
         let output_fd = open(output, ...);//output redirection -- A subprocess
         close(1); //Close file descriptor 1
         dup(output_fd);//File descriptor 1 and file descriptor output_fd point to the same file
         close(output_fd);//Close the file descriptor output_fd
     //Execute the new program after I/O redirection
      exec(args_copy[0].as_str(), args_addr.as_slice());
     }...
```

<!-- ![bg right:30% 100%](figs/user-stack-cmdargs.png) -->



---
<style scoped>
{
  font-size: 30px
}
</style>

**Outline**

1. Experimental arrangement
2. Code structure
3. Pipeline design and implementation
### 4. Signal design and implementation
- Signal system call
- Signal core data structure
- Build signal_handler
- Support kill system call

![bg right:40% 95%](figs/ipc-os-detail-2.png)

---

#### System calls related to signal processing

<!-- https://www.onitroad.com/jc/linux/man-pages/linux/man2/sigreturn.2.html -->
- Sigaction: set signal processing routine
- Sigprocmask: set the signal to block
- Kill: send a signal to a process
- Sigreturn: clear stack frame, return from signal handling routine

![bg right:50% 100%](figs/signal-process.png)

---
#### System calls related to signal processing
<!-- https://www.onitroad.com/jc/linux/man-pages/linux/man2/sigreturn.2.html -->
```rust
// Set up the signal handling routine
// signum: specify the signal
// action: new signal processing configuration
// old_action: old signal processing configuration
sys_sigaction(signum: i32,
    action: *const SignalAction,
    old_action: *const SignalAction)
    -> isize

pub struct SignalAction {
     // Address of the signal handler routine
     pub handler: usize,
     // signal mask
     pub mask: SignalFlags
}
```

![bg right:50% 100%](figs/signal-process.png)


---
#### System calls related to signal processing
<!-- https://www.onitroad.com/jc/linux/man-pages/linux/man2/sigreturn.2.html -->
```rust
// set the signal to block
// mask: signal mask
sys_sigprocmask(mask: u32) -> isize
```
```rust
// Clear stack frame, return from signal handling routine
  sys_sigreturn() -> isize
```
```rust
// Send a signal to a process
// pid: process pid
// signal: the integer code of the signal
sys_kill(pid: usize, signal: i32) -> isize
```
![bg right:50% 100%](figs/signal-process.png)


---
#### The core data structure of the signal
Signal core data structure in process control block
```rust
pub struct TaskControlBlockInner {
     ...
     pub signals: SignalFlags, // Signals to respond to
     pub signal_mask: SignalFlags, // signal to be masked
     pub handling_sig: isize, // signal being processed
     pub signal_actions: SignalActions, // signal processing routine table
     pub killed: bool, // Whether the task has been killed
     pub frozen: bool, // Whether the task has been suspended
     pub trap_ctx_backup: Option<TrapContext> //interrupted trap context
}
```
<!--
The function of killed is to mark whether the current process has been killed. Because the process will not end immediately when it receives the kill signal, but will exit at an appropriate time. At this time, killed is needed as a mark to exit the unnecessary signal processing loop.

The frozen flag is related to the two signals SIGSTOP and SIGCONT. SIGSTOP will suspend the execution of the process, that is, set frozen to true. At this time, the current process will block waiting for SIGCONT (that is, the unfreezing signal). When the signal receives SIGCONT, set frozen to false, exit the cycle of waiting for the signal, and return to user mode to continue execution. -->

---
#### Create signal_handler

```rust
fn sys_sigaction(signum: i32, action: *const SignalAction,
                           old_action: *mut SignalAction) -> isize {
   //Save the old signal_handler address to old_action
   let old_kernel_action = inner.signal_actions.table[signum as usize];
   *translated_refmut(token, old_action) = old_kernel_action;
  //Set the new signal_handler address to TCB's signal_actions
   let ref_action = translated_ref(token, action);
   inner.signal_actions.table[signum as usize] = *ref_action;
```
For the signal number signum that needs to be modified:
1. Save the old signal_handler address to `old_action`
2. Set `action` to the new signal_handler address



---
#### Signal via kill

```rust
fn sys_kill(pid: usize, signum: i32) -> isize {
       let Some(task) = pid2task(pid);
       // insert the signal if legal
       let mut task_ref = task. inner_exclusive_access();
       task_ref.signals.insert(flag);
      ...
```
Send a signal with value `signum` to the process whose process ID is `pid`:
1. Find TCB according to `pid`
2. Insert the `signum` signal value in the signals in the TCB



---
#### The process of sending and processing signals through kill
When the `pid` process enters the kernel, the execution process before returning to the user mode from the kernel:
```
Execute APP --> __alltraps
          --> trap_handler
             --> handle_signals
                 --> check_pending_signals
                     --> call_kernel_signal_handler
                     --> call_user_signal_handler
                        --> // backup trap Context
                             // modify trap Context
                             trap_ctx.sepc = handler; //Set the entry back to the interrupt processing routine
                             trap_ctx.x[10] = sig; // put the signal value in Reg[10]
             --> trap_return //Find and jump to the `__restore` assembly function located on the springboard page
        --> __restore //Restore the modified trap Context, execute sret
Execute the signal_handler function of APP
  ```

 
---
#### APP resumes normal execution
When the process whose process ID is pid finishes executing the main body of the signal_handler function, it will issue the `sys_sigreturn` system call:
```rust
fn sys_sigreturn() -> isize {
   ...
   // Restore the previously backed up trap context
   let trap_ctx = inner. get_trap_cx();
   *trap_ctx = inner.trap_ctx_backup.unwrap();
   ...
Execute APP --> __alltraps
        --> trap_handler
             --> Handle the sys_sigreturn system call
             --> trap_return //Find and jump to the `__restore` assembly function located on the springboard page
     --> __restore //Restore the modified trap Context, execute sret
Where the execution of the APP is interrupted
```


---
#### Shield signal
```rust
fn sys_sigprocmask(mask: u32) -> isize {
     ...
     inner.signal_mask = flag;
     old_mask. bits() as isize
     ...
```
Record the signal to be masked directly into the signal_mask data of TCB

---
### Summary
- The concept and implementation of the pipeline
- Signal concept and implementation
- Can write the Velociraptor operating system

![bg right 80%](figs/velociraptor.png)

---

### Course experiment four file system and inter-process communication

* Chapter 6: File system and I/O redirection -> chapter6 exercises ->
     * [rCore](https://learningos.github.io/rCore-Tutorial-Guide-2022A/chapter6/4exercise.html#id1)
     * [uCore](https://learningos.github.io/uCore-Tutorial-Guide-2022A/chapter6/5exercise.html#id3)
* Experimental tasks
     * Hard link
* Experiment submission requirements
     * The 11th day after the task is assigned (December 04, 2022);