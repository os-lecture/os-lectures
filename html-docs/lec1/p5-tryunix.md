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

## Lecture 1 Operating System Overview

### Section 5 Practice: Try UNIX/Linux

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---
## Where is UNIX/Linux?

- Linux
    - Ubuntu, Fedora, SuSE, openEuler
    - Kirin Tongxin
- Windows with WSL (Windows Subsystem of Linux)
-MacOS with UNIX shell
---
## Why Linux?
- Open source, well documented, clean design, widely used
- Learning ucore/rcore will help if you know Linux internals.

---
## Try Linux

-shell
    - bash, fish, zsh, starship...

- program
    - ls, rm, gcc, gdb, vim...

---
## What services does the Linux kernel usually provide?

   * Process (a running program)
   * Memory allocation
   * Document content
   * File name, directory
   * Access Control (Security)
   * Many others: user, IPC, network, time, endpoint


---
## The application/kernel interface provided by the Linux kernel?

   * "System call"
   * Examples, in C, from UNIX (e.g. Linux, macOS, FreeBSD).

             fd = open("out", 1);
             write(fd, "hello\n", 6);
             pid = fork()

  * These look like function calls, but they are not
  * The number of core system calls is not large (about 20)

---
## The application/kernel interface provided by the Linux kernel?

<style scoped>
table {
  font-size: 25px;
}
</style>

| System call name | Meaning |
| ------------------------ | ---- |
| ``int fork()`` | Create a process and return the PID of the child process. |
| ``int exit(int status)`` | Terminate the current process; report the status to the parent process that executes the wait() system call. Nothing is returned. |
| ``int wait(int *status)`` | Wait for the child process to exit; the exit status is ``*status``; return the PID of the child process. |
| ``int kill (int pid)`` | Terminate the process whose process ID is PID. Returns 0 for success, or -1 for error. |
| ``int getpid()`` | Returns the PID of the current process. |

---
## The application/kernel interface provided by the Linux kernel?

<style scoped>
table {
  font-size: 28px;
}
</style>

| System call name | Meaning |
| ------------------------ | ---- |
| ``int sleep(int n)`` | Pause for n clock cycles. |
| ``int exec(char *file, char *argv[])`` | Load file with arguments and execute; return only on error. |
| ``char *sbrk(int n)`` | Increase process memory by n bytes. Returns the start address of the new memory. |
| ``int open(char *file, int flags)`` | Open the file; the flag indicates the read/write attribute of the file operation; return a fd (file descriptor). |
| ``int write(int fd, char *buf, int n)`` | Write n bytes from buf to file descriptor fd; return n. |

---
## The application/kernel interface provided by the Linux kernel?

<style scoped>
table {
  font-size: 28px;
}
</style>

| System call name | Meaning |
| ------------------------ | ---- |
| ``int read(int fd, char *buf, int n)`` | Read n bytes into buf; return the number read, or 0 if end-of-file. |
| ``int close(int fd)`` | Release the open file with descriptor fd. |
| ``int dup(int fd)`` | Returns a new file descriptor referencing the same file as the file descriptor. |
| ``int pipe(int p[])`` | Creates a pipe with read/write file descriptors in p[0] and p[1]. |
| ``int chdir(char *dir)`` | Change the current directory. |

---
## The application/kernel interface provided by the Linux kernel?

<style scoped>
table {
  font-size: 28px;
}
</style>

| System call name | Meaning |
| ------------------------ | ---- |
| ``int mkdir(char *dir) `` | Create a new directory. |
| ``int mknod(char *file, int, int)`` | Create a device file. |
| ``int fstat(int fd, struct stat *st)`` | put the meta information of file fd into ``*st`` |
| ``int stat(char *file, struct stat *st)`` | Put the meta information of the file ``*file`` into ``*st`` |
| ``int link(char *file1, char *file2)`` | Create another name (file2) for the file file1. |
| ``int unlink(char *file)`` | Delete a file. |


---
## Analyze UNIX/Linux applications

[Analysis of some very simple small programs](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/)

#### Process related

fork.c exec.c forkexec.c ...
#### File system related
list.c open.c echo.c copy.c ...
#### Inter-process communication related
  pipe1.c pipe2.c redirect.c ...

---
## Analyze UNIX/Linux applications
  For example: [copy.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/copy.c), copy input to output
read bytes from input, write them to output

         $ copy

   copy.c is written in C language
    
   read() and write() are system calls
   The first parameter of read()/write() is "file descriptor" (fd)
   Passed to the kernel to tell it which "open file" to read/write.

---
## Analyze UNIX/Linux applications

<style scoped>
{
  font-size: 30px
}
</style>

An FD (descriptor) that must have been previously opened is connected to a file/device/socket
A process can have many files open, with many descriptors
UNIX convention: FD: 0 is "standard input", 1 is "standard output"


The second parameter of read() is a pointer to some memory to be read.

The third parameter of read() is the maximum number of bytes to read

Note: read() can read less, but not more


---
## Analyze UNIX/Linux applications

Return value: the actual number of bytes read, or -1 for an error
Note: copy.c doesn't care about the format of the data.
UNIX I/O is 8-bit bytes
Interpretation is application specific, e.g. database records, C source code, etc.
Where do file descriptors come from?


---
## Analyze UNIX/Linux applications

For example: [open.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/open.c), create a file

     $ open
     $ cat output.txt

open() creates a file, returning a file descriptor (or -1 on error).
FD is a small integer, and FD is indexed into a per-process table maintained by the kernel

Different processes have different FD namespaces. For example, FD 1 means different things to different processes

Further details can be found in UNIX manuals, eg "man 2 open".
man 1 is a shell command such as ls; man 2 is a system call such as open; man 3 is a function description

---
## Analyze UNIX/Linux applications

What happens when a program calls a system call like open()?

- looks like a function call, but it's actually a special instruction
- The hardware saves some user registers
- Hardware elevated privilege level
- hardware jump to a known "entry point" in the kernel
- Now run C code in the kernel


---
## Analyze UNIX/Linux applications

What happens when a program calls a system call like open()?

- Kernel call system call implementation
- open() looks up the name in the filesystem
- it may wait for the disk to arrive
- Update kernel data structures (cache, FD table)
- restore user registers
- Lower the permission level
- Jump back to the calling point in the program and continue running
- We will see more details in later lessons

---
## Analyze UNIX/Linux applications

Input information to the UNIX command line interface (shell).
The shell prints a "$" prompt.
The shell lets you run UNIX command-line tools
Useful for system administration, working with files, developing and writing scripts

     $ ls
     $ ls > out
     $ grep x < out

---
## Analyze UNIX/Linux applications

However, supporting time-sharing and multi-tasking execution through the shell is the focus of UNIX design at the beginning.
Many system calls can be made through the shell.

The shell creates a new process for each command you type, for example, for

     $ echo hello



---


## Analyze UNIX/Linux applications
The fork() system call creates a new process

     $ fork

The kernel creates a copy of the calling process
- Instructions, data, registers, file descriptors, current directory
- "parent" and "child" processes

---


## Analyze UNIX/Linux applications

The only difference: fork() returns a pid in the parent process and 0 in the child process.
pid (process ID) is an integer, the kernel gives each process a different pid

Therefore, the "fork() returns" of [fork.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/fork.c) in **two** processes will execute
"if(pid == 0) " realizes the distinction between parent and child processes

---
## Analyze UNIX/Linux applications

How can we run a new program in this process?

For example: [exec.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/exec.c), replace the calling process with an executable file.
How the shell runs a program such as

     $ echo a b c

A program is stored in a file: instructions and initial memory, created by the compiler and linker
So there is a file called echo, which contains the operation commands for the `exec` system call

---
## Analyze UNIX/Linux applications

exec() replaces the current process with an executable file
- Discard instruction and data memory
- load instructions and memory from file
- File descriptors are preserved

---
## Analyze UNIX/Linux applications

exec(filename, argument-array)
argument-array holds command line arguments; exec passed to main()

     cat user/echo.c

echo.c shows how a program sees its command line arguments

---
## Analyze UNIX/Linux applications

For example: [forkexec.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/forkexec.c), fork() a new process, exec() a program.

       $ forkexec

forkexec.c contains a common UNIX idiom.
- fork() a child process
- exec() a command in a child process
- The parent process waits for the child process to complete


---
## Analyze UNIX/Linux applications

The shell forks/execs/waits every command you enter.
After wait(), the shell will print out the next prompt
Run in the background -- `&` -- , the shell will skip wait()


---
## Analyze UNIX/Linux applications

exit(status) --> wait(&status)

status convention: 0 = success, 1 = command encountered an error
Note: fork() makes a copy, but exec() discards the copied memory.
This may seem wasteful.
You can transparently eliminate copying with "copy-on-write" techniques.


---
## Analyze UNIX/Linux applications

Example: [redirect.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/redirect.c), redirects the output of a command
What does the shell do about this?

     $ echo hello > out
 
Answer: fork, change the FD1 of the child process, execute echo

     $ redirect
     $ cat output.txt

---
<style scoped>
{
  font-size: 30px
}
</style>

## Analyze UNIX/Linux applications

Note: open() always chooses the lowest unused FD; the choice of 1 is due to close(1).
fork, FD and exec interact nicely for I/O redirection
Independent fork-then-exec gives the child process a chance to change the FD before exec.
FDs provide indication
Commands only need to use FDs 0 and 1, no need to know their positions
exec preserves the FDs set by sh
Therefore: only sh needs to know about I/O redirection, not every program



---
## Analyze UNIX/Linux applications

Some questions to ponder:
- Why these I/O and process abstractions? Why not something else?
- Why provide a filesystem? Why not let programs use the disk in their own way?
-Why FDs? Why not pass a filename to write()?
- Why are files streams of bytes, not disk blocks or formatted records?
- Why not combine fork() and exec()?

The UNIX design works well, but we'll see other designs

---
## Analyze UNIX/Linux applications

Example: [pipe1.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/pipe1.c), communicate through a pipe
How the shell is implemented

     $ ls | grep x
     $pipe1

An FD can refer to either a "pipe" or a file.
The pipe() system call creates two FDs
- read from the first FD
- write to the second FD
  

---
## Analyze UNIX/Linux applications

The kernel maintains a buffer for each pipe
- write() adds to the buffer
- read() waits until data appears

---
## Analyze UNIX/Linux applications

Example: [pipe2.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/pipe2.c), communicates between processes.
Pipes combine nicely with fork() to do ls|grep x.
The shell creates a pipe.
Then fork (twice).
Then connect the FD1 of ls to the write FD of the pipe.
and grep's FD 0 connected to the pipe.

    $ pipe2 -- a simplified version

Pipes are a separate abstraction, but combine nicely with fork()


---
## Analyze UNIX/Linux applications


* Example: [list.c](https://pdos.csail.mit.edu/6.828/2021/lec/l-overview/list.c), list files in a directory
How does ls get a list of files in a directory?
You can open a directory and read it -> filename
"... " is a pseudonym for the current directory of a process
See ls.c for more details

---
## Analyze UNIX/Linux applications

summary

   * We have studied UNIX's I/O, file system and process abstractions
   * These interfaces are very simple, only integers and I/O buffers
   * These abstractions combine well, for example, I/O redirection