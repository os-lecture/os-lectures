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

# Lecture 3 Isolation and batch processing based on privilege level
## The second section looks at RISC-V from the perspective of OS

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---

**Outline**

### 1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming

---
#### The main goal of this section

- Understand RISC-V privilege levels and hardware isolation methods
- Understand the basic characteristics of RISC-V's M-Mode and S-Mode
- Understand how the OS **accesses and controls** computer systems in M-Mode and S-Mode
- Learn how different software can **switch between M-Mode<–>S-Mode<–>U-Mode**
---
#### Mainstream CPU Comparison
<!-- Mainly explain that x86, arm due to compatibility and historical reasons, lead to complex design and implementation, riscv is simple/flexible/extensible, easy to learn and master and used to write OS -->
![w:1150](figs/mainstream-isas.png)

---
#### Mainstream CPU Comparison
* Due to compatibility and historical reasons, the design and implementation of x86 and ARM are complicated
* RISC-V is concise/flexible/extensible

![w:1150](figs/x86-arm-rv-compare.png)


---
**Outline**

1. Mainstream CPU Comparison
### 2. RISC-V system mode
   - overview
   - Privileged
   - CSR register
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming

---

#### RISC-V related terms
- Application Execution Environment (AEE)
- Application Binary Interface (ABI)
- Supervisor Binary Interface (SBI)
- Supervisor Execution Environment (SEE)
- Hypervisor: virtual machine monitor
-Hypervisor Binary interface (HBI)
- Hypervisor Execution Environment (HEE)


---
<style scoped>
{
  font-size: 30px
}
</style>

#### RISC-V system mode
![w:800](figs/rv-privil-arch.png)
-ABI/SBI/HBI:Application/Supervisor/Hypervisor Bianry Interface
-AEE/SEE/HEE:Application/Superv/Hyperv Execution Environment
- HAL: Hardware Abstraction Layer
- Hypervisor, virtual machine monitor (virtual machine monitor, VMM)
- RISC-V system mode is the RISC-V mode related to system programming



---
<style scoped>
{
  font-size: 30px
}
</style>

#### RISC-V system mode: single application scenario
![w:800](figs/rv-privil-arch.png)
- Different software layers have clear privilege-level hardware isolation support
- The **single app** on the left is coded to run on the ABI
- ABI is the interface for interaction between user-level ISA (Instruction Set Architecture) and AEE
- ABI hides the details of AEE from the application, making AEE more flexible

---
<style scoped>
{
  font-size: 30px
}
</style>

#### RISC-V System Mode: Operating System Scenarios
![w:800](figs/rv-privil-arch.png)
- A **traditional operating system** is added in the middle to support multi-programming of multiple applications
- Each application communicates with the OS via **ABI**
- The RISC-V operating system communicates with SEE through **SBI**
- SBI is the interface for the interaction between the OS kernel and SEE, and supports the ISA of the OS

---
#### RISC-V system mode: virtual machine scenario
![w:800](figs/rv-privil-arch.png)
- The virtual machine scene on the right can support **multiple operating systems**



---
#### RISC-V System Mode: Application Scenarios
![w:800](figs/rv-privil-arch.png)
- M Mode: small devices (bluetooth headsets, etc.)
- U+M Mode: Embedded device (TV remote control, credit card machine, etc.)
- U+S+M Mode: mobile phone
- U+S+H+M Mode: data center server

---
<style scoped>
{
  font-size: 28px
}
</style>

#### RISC-V System Mode: Hardware Threading
![w:800](figs/rv-privil-arch.png)
- Privileged level is a protection mechanism provided for different software stack components
- **Hardware thread** (hart, ie CPU core) is running at a privileged level (CSR configuration)
- When the processor performs an operation that is not allowed by the current privileged mode, an **exception** will be generated. These exceptions usually generate a trap (trap) that causes the **lower-level execution environment to take over control**

---
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
   - overview
   - **Privileged**
   - CSR register
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming

---

#### RISC-V system mode: multiple privilege levels
![w:800](figs/rv-privil-arch.png)
-Modern processors generally have multiple privileged modes (Mode)
- **U**: User | **S**: Supervisor | **H**: Hypervisor | **M**: Machine

Why are there these **4 modes**? What are their **differences and connections**?




---
#### RISC-V System Mode: Execution Environment
| Execution Environment | Encoding | Meaning | Across Privilege Levels |
| --- | --- | --------------------- | --- |
| APP | 00 | User/Application | ``ecall`` |
| OS | 01 | Supervisor | ``ecall`` ``sret`` |
| VMM | 10 | Hypervisor | --- |
| BIOS | 11 | Machine | ``ecall`` ``mret`` |

- The combined hardware system of M, S, U is suitable for running UNIX-like operating systems


---
#### RISC-V system mode: flexible combination of privilege levels
![w:800](figs/rv-privil-arch.png)
- As the **requirements of the application change**, a **flexible** and **combinable** hardware structure is required
- Therefore, the above four modes appeared, and the flexible hardware design that can be combined between the modes

---
#### RISC-V system mode: user mode
![w:800](figs/rv-privil-arch.png)
- U-Mode (User Mode, user mode, user mode)
   - **Unprivileged** level mode (Unprivileged Mode): basic computing
   - It is the user-mode CPU execution mode of **application running**
   - Cannot execute privileged instructions and cannot directly affect the execution of other applications


---
<style scoped>
{
  font-size: 28px
}
</style>
#### RISC-V system mode: kernel mode
![w:800](figs/rv-privil-arch.png)
- S-Mode (Supervisor Mode, Kernel Mode, kernel mode, kernel mode)
   - The operating system in the kernel mode has sufficient **hardware control capability**
   - Privileged Mode: **Limit APP** execution and memory access
   - It is the kernel state CPU execution mode of **operating system running**
   - Can execute kernel state privileged instructions, can directly **affect application program execution**

---
<style scoped>
{
  font-size: 30px
}
</style>
#### RISC-V system mode: H-Mode
![w:800](figs/rv-privil-arch.png)
- H-Mode (Hypervisor Mode, Virtual Machine Mode, virtual machine monitor)
   - Privileged mode: **Limit the memory space accessed by the OS**
   - It is the Hypervisor Mode CPU execution mode of **virtual machine monitor operation**, which can execute H-Mode privileged instructions and can directly affect OS execution**


---
<style scoped>
{
  font-size: 30px
}
</style>
#### RISC-V system mode: M-Mode
![w:800](figs/rv-privil-arch.png)
- M-Mode (Machine Mode, Physical Machine Mode)
   - Privileged mode: **control physical memory**, directly shut down
   - It is the Machine Mode CPU execution mode of **Bootloader/BIOS running**
   - Ability to execute M-Mode privileged instructions, which can directly affect the execution of other software mentioned above

---
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
   - overview
   - Privileged
   - **CSR register**
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming

---

#### RISC-V CSR register classification

- **General purpose registers** x0-x31
   - General command access
   - Fastest memory location available to unprivileged instructions
- **Control Status Register** (CSR: Control and Status Registers)
   - Accessed via the **Control Status Register Instruction**, there can be 4096 CSRs
   - Applications running in **User Mode** cannot access most of the CSR registers
   - The operating system running in **kernel mode** controls the computer by accessing the CSR register

<!---
## RISC-V system mode: control status register CSR
Mandatory isolation to avoid availability/reliability/security impact on the entire system -->
---
<style scoped>
{
  font-size: 32px
}
</style>
#### Isolation via CSR register
The OS ensures the safety and reliability of the computer through hardware isolation (three defenses)
- Set CSR (Control Status Register) to achieve isolation
   - Power: prevent applications from accessing system control related registers
     - **ADDRESS SPACE CONFIGURATION** register: mstatus/sstatus CSR
   - Time: Prevent apps from using 100% CPU for long periods of time
     - **Interrupt configuration** register: sstatus/stvec CSR
   - Data: Prevent app damage from stealing data
     - **Address space related** registers: sstatus/satp/stvec CSR

<!---
## RISC-V 系统模式：控制状态寄存器CSR

- mtvec(MachineTrapVector)保存发生异常时需要跳转到的地址。
- mepc(Machine Exception PC)指向发生异常的指令。
- mcause(Machine Exception Cause)指示发生异常的种类。
- mie(Machine Interrupt Enable)指出处理器目前能处理的中断。
- mip(Machine Interrupt Pending)列出目前正准备处理的中断。
- mtval(Machine Trap Value)保存陷入(trap)附加信息:地址例外中出错的地址、发生非法指令例外的指令本身；对于其他异常，值为0。
- mscratch(Machine Scratch)它暂时存放一个字大小的数据。
- mstatus(Machine Status)保存全局中断以及其他的状态
-->

<!-- 
---
## RISC-V 系统模式：控制状态寄存器CSR

- mcause(Machine Exception Cause)它指示发生异常的种类。
  - SIE控制S-Mode下全局中断，MIE控制M-Mode下全局中断。
  - SPIE、MPIE记录发生中断之前MIE和SIE的值。
---
## RISC-V 系统模式：控制状态寄存器CSR

- sstatus(supervisor status)保存发生异常时需要跳转到的地址。
- stvec(supervisor trap vector)保存s模式的trap向量基址。stvec总是4字节对齐
- satp(supervisor Address Translation and Protection) S-Mode控制状态寄存器控制了分页系统。
- sscratch (supervisor Scratch Register) 保存指向hart-local supervisor上下文的指针. 在trap处理程序的开头，sscratch与用户寄存器交换，以提供初始工作寄存器。
- sepc(supervisor Exception PC)它指向发生异常的指令。
-->

---
<style scoped>
{
  font-size: 32px
}
</style>

**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
### 3. RISC-V system programming: user mode programming
   - brief description
   - U-Mode programming: system calls
   - Privileged operations
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming

---

#### System Programming Brief
- System programming needs to understand the **privileged level architecture** of the processor, and be familiar with the register resources, memory resources and peripheral resources that can be accessed by each privileged level
- **Write kernel-level code**, construct an operating system, and support application execution
   - Memory management process scheduling
   - Exception handling Interrupt handling
   - System calls Peripheral control
- System programming is usually **not supported by **extensive user** programming libraries** and convenient dynamic **debugging tools**
- The system programming of this course mainly focuses on S-Mode and U-Mode of RISC-V, involving some understanding of M-Mode

---
#### RISC-V U-Mode programming: using system calls
- Applications under U-Mode cannot directly use the physical resources of the computer
- Environment call exception: Occurs when executing ``ecall``, which is equivalent to a system call
- OS can directly access physical resources
- What if the application needs to use hardware resources?
   - print "hello world" to the screen
   - read data from file
- Obtain services from the operating system through system calls

---
#### U-Mode programming: first example "hello world"
[Small example of printing "hello world" in user mode](https://github.com/chyyuu/os_kernel_lab/tree/v4-kernel-sret-app-ecall-kernel/os/src) roughly execution flow

![w:1000](figs/print-app.png)


---
#### Start execution of the first example
[A small example of printing "hello world" in user mode](https://github.com/chyyuu/os_kernel_lab/blob/v4-kernel-sret-app-ecall-kernel/os/src/main.rs#L302) Start the execution flow

![w:1000](figs/boot-print-app.png)



---
#### The second example: Executing privileged instructions in user mode
[A small example of executing privileged instructions in user mode](https://github.com/chyyuu/os_kernel_lab/blob/v4-illegal-priv-code-csr-in-u-mode-app-v2/os/src/ main.rs#L306) Startup and execution process

![w:1000](figs/boot-priv-code-app.png)


<!-- Zifencei extension https://www.cnblogs.com/mikewolf2002/p/11191254.html -->
---
#### Privileged Operations
- Privileged operations: privileged instructions and CSR read and write operations
- Very few instructions:
   - ``mret`` machine mode return
   - ``sret`` supervisor mode return
   - ``wfi`` wait for interrupt
   - ``sfence.vma`` virtual address barrier (barrier) instruction
  
- Many other system management functions are implemented by reading and writing the control status register

Note: ``fence.i`` is an i-cache barrier (barrier) instruction, a non-privileged instruction, which belongs to the "Zifencei" extended specification

<!-- Before executing the fence.i instruction, for the same hardware thread (hart), RISC-V does not guarantee that the data written to the memory instruction area by the store instruction can be fetched by the fetch instruction. After using the fence.i command, for the same hart, you can ensure that the command reads the data that was written to the memory command area recently. However, fence.i will not guarantee that the reading of other riscv hart instructions can also meet the read and write consistency. If you want to make the write instruction memory space meet the consistency requirements for all harts, you need to execute the fence instruction. -->


---
<style scoped>
{
  font-size: 32px
}
</style>
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
### 4. RISC-V system programming: M-Mode programming
   - Interrupt mechanism and exception mechanism
   - Hardware response to interrupt/exception
   - Handover of control for interrupt/exception handling
5. RISC-V system programming: kernel programming

---
<style scoped>
{
  font-size: 30px
}
</style>
#### M-Mode programming
- M-Mode is the **highest authority mode** of hart (hardware thread) in RISC-V
- In M-Mode, hart has **full access to the underlying functions of the computer system**
- The most important feature of M-Mode is **intercept and handle interrupt/exception**
   - **Synchronous exception**: generated during execution, accessing an invalid register address, or executing an instruction with an invalid opcode
   - **Asynchronous Interrupt**: Instruction flow asynchronous external events, interrupts, such as clock interrupts
- RISC-V requires the implementation of **precise exception**: to ensure that all instructions before the exception are fully executed, and subsequent instructions have not started to execute

---
<style scoped>
{
  font-size: 30px
}
</style>
#### M-Mode interrupt control and status register

- mtvec(MachineTrapVector) saves the **interrupt processing routine entry address** to be jumped to when an interrupt/exception occurs
- mepc(Machine Exception PC) points to the **instruction when an interrupt/exception occurred**
- mcause(Machine Exception Cause) indicates the type of **interrupt/exception** that occurred
- mie(Machine Interrupt Enable) interrupt **enable** register
- mip (Machine Interrupt Pending) interrupt **request** register
- mtval(Machine Trap Value) saves trapped (trap) **Additional Information**
- mscratch (Machine Scratch) it temporarily stores a word-sized **data**
- mstatus(Machine Status) saves global interrupts and other **status**

<!-- mtval (Machine Trap Value) saves the additional information of the trap: the address of the error in the address exception, the instruction itself where the exception of the illegal instruction occurred; for other exceptions, the value is 0. -->
---
<style scoped>
{
  font-size: 28px
}
</style>
#### Mstatus CSR register

- mstatus(Machine Status) saves global interrupts and other **status**
   - SIE controls the global interrupt in S-Mode, and MIE controls the global interrupt in M-Mode.
   - SPIE, MPIE records the values of MIE and SIE before the interruption occurred.
   - SPP indicates whether the privilege level before the change is S-Mode or U-Mode
   - MPP indicates whether it was S-Mode, U-Mode or M-Mode before the change
   PP: Previous Privilege


![w:1000](figs/mstatus.png)


---
#### Mcause CSR register

When an exception occurs, a code indicating **the event that caused the exception** is written in the mcause CSR. If the event is caused by an interrupt, the ``Interrupt`` bit is set, and the ``Exception Code`` field contains the indication of the last An unusual encoding.

![w:1150](figs/rv-cause.png)

---
#### M-Mode Clock Interrupt Timer
- Interrupts happen asynchronously
   - Signals from I/O devices external to the processor
- Timer can generate interrupts in a stable and regular manner
   - Prevent the application program from occupying the CPU, so that the OS Kernel can get the execution right...
   - CPU control by **software in high privilege mode**
   - Software in high privilege mode can **authorize** low privilege mode software to handle interrupts

---
#### RISC-V processor FU540 module diagram
![w:650](figs/fu540-top-block.png)

---
<style scoped>
{
  font-size: 32px
}
</style>
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
   - Interrupt mechanism and exception mechanism
### Interrupt/exception hardware response
   - Handover of control for interrupt/exception handling
5. RISC-V system programming: kernel programming

---
<style scoped>
{
  font-size: 25px
}
</style>
#### M-Mode interrupt hardware response process
- The PC of the **abnormal instruction** is saved in mepc, and the PC is set to mtvec.
   - For synchronous exceptions, mepc points to the instruction that caused the exception;
   - For interrupts, points to where execution should resume after interrupt handling.
- Set mcause according to **exception source**, and set mtval to error address or other information word suitable for **specific exception**
- Set mstatus[MIEbit] to zero to **disable interrupts** and **keep previous MIE value** into MPIE
     - SIE controls the global interrupt in S mode, MIE controls the global interrupt in M mode;
     - SPIE records the value before SIE interruption, and MPIE records the value before MIE interruption
- **Reserve the permission mode before the exception occurred** to the MPP domain of mstatus, and then **change the permission mode** to M. (MPP indicates that the privilege level before the change is S, M or U mode)

---
#### M-Mode interrupt handling routine
```
     let cause = cause::read();
     let stval = stval::read();

     match cause. cause() {
         Trap::Exception(Exception::UserEnvCall) => {
             cx.sepc += 4;
             cx.x[10] = do_syscall(cx.x[17], [cx.x[10], cx.x[11], cx.x[12]]) as usize;
         }
         _ => {
             panic!(
                 "Unsupported trap {:?}, stval = {:#x}!",
                 cause. cause(),
                 stval
             );
         }
     }

```
---

#### M-Mode interrupt classification
The type of interrupt is obtained by different bits (mie) of the mcause register.
- **Software** interrupt: Triggered by writing data to a memory-mapped register, one hart interrupts another hart (interprocessor interrupt)
- **Clock** interrupt: hart's time counter register mtime is greater than time comparison register mtimecmp
- **External** interrupt: Triggered by the interrupt controller, in most cases the peripherals will be connected to this interrupt controller

---
#### RISC-V Interrupts/Exceptions
The information of the interrupt source is obtained through different bits of the mcause register.
The first column 1 represents the interrupt, the second column represents the interrupt ID, and the third column represents the interrupt meaning
![w:1000](figs/rv-interrupt.png)


---

#### M-Mode RISC-V exception mechanism
The information that caused the exception is obtained through different bits of the mcause register.
The first column 0 represents the exception, the second column represents the exception ID, and the third column represents the meaning of the exception
![bg right:45% 100%](figs/rv-exception.png)

---
#### M-Mode interrupt/abnormal hardware response
- The PC of the interrupt/exception instruction is stored in mepc, and the PC is set to mtvec.
    - For exceptions, mepc points to the instruction that caused the exception
    - For interrupts, mepc points to where execution should resume after interrupt handling
- Set mcause according to the **interrupt/exception source** and set mtval to the address of the error or other information word suitable for the specific exception.

---
<style scoped>
{
  font-size: 30px
}
</style>
#### Handover of M-Mode interrupt/exception handling
- **mideleg/medeleg** (Machine Interrupt/Exception Delegation) CSR controls which interrupts/exceptions are delegated to S-Mode for processing
- Each of mideleg/medeleg corresponds to an interrupt/exception
   - If mideleg[5] corresponds to the S-Mode clock interrupt, if it is set, the S-Mode clock interrupt will be handed over to the S-Mode interrupt/exception handler instead of the M-Mode interrupt/exception handler program
   - Any interrupts delegated to S-Mode can be masked by S-Mode software. sie (Supervisor Interrupt Enable) and sip (Supervisor Interrupt Pending) CSR are S-Mode control status registers

---

#### Interrupt delegation register mideleg
- Mideleg (Machine Interrupt Delegation) controls which interrupts are delegated to S-mode processing
- Each bit in mideleg corresponds to an interrupt/exception
   - mideleg[1] is used to control whether **inter-core interrupt** is handed over to s mode for processing
   - mideleg[5] is used to control whether to hand over **timing interrupt** to s mode for processing
   - mideleg[9] is used to control whether to hand over **external interrupt** to s mode for processing


---

#### Exception delegation register medeleg
- Medeleg (Machine Exception Delegation) controls which exceptions are delegated to S mode for processing
- Each in the medeleg corresponds to an interrupt/exception
   - medeleg[1] is used to control whether to hand over the **instruction acquisition error exception** to the s mode for processing
   - medeleg[12] is used to control whether to hand over **instruction page exception** to s mode for processing
   - medeleg[9] is used to control whether **data page exception** is handed over to s mode for processing

<!-- , is a subset of mie and mip. These two registers have the same layout as in M-Mode. In sie and sip, only bits corresponding to interrupts delegated by mideleg can be read and written, and bits corresponding to interrupts not delegated are always 0 -->

---
#### Handover of control for interrupt/exception handling


- When an interrupt/exception occurs, processor control **usually** will not be handed over to a less privileged mode
   - eg medeleg[15] will delegate store page fault to S-Mode
   - Exceptions that occur in M-Mode are always handled in M-Mode
   - Exceptions in S-Mode are always handled in M-Mode, or in S-Mode
   - Exceptions that occur in the above two modes will not be handled by U-Mode
  
**Why?**


---
#### Thinking questions

- How to implement the breakpoint debugging function of the debugger through the breakpoint exception?
- How to implement single-step tracking?
---
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
### 5. RISC-V system programming: kernel programming
   - Interrupt/exception mechanism
   - Interrupt/exception handling
   - Virtual memory mechanism
---
<style scoped>
{
  font-size: 30px
}
</style>
#### S-Mode interrupt control and status register

- stvec(SupervisorTrapVector) saves the address **to jump to when an interrupt/exception occurs**
- sepc(Supervisor Exception PC) points to the **instruction when an interrupt/exception occurred**
- cause(Supervisor Exception Cause) indicates the **kind** of the interrupt/exception that occurred
- sie(Supervisor Interrupt Enable) interrupt **enable** register
- sip (Supervisor Interrupt Pending) interrupt **request** register
- stval(Supervisor Trap Value) saves trap (trap) **Additional Information**
- sscratch (Supervisor Scratch) different mode exchange **data transfer station**
- sstatus (Supervisor Status) saves global interrupts and other **status**

---
#### Sstatus register
- The SIE and SPIE bits of sstatus respectively save the current and interrupt enable **status** before the interrupt/exception occurs

![w:1100](figs/rv-sstatus.png)

---
#### S-Mode Interrupt/Exception Mechanism

**sie & sip register** is a CSR used to save **pending interrupt** and **interrupt enable**

- sie (supervisor interrupt-enabled register)
- sip (supervisor interrupt pending)

![w:1150](figs/rv-sie-sip.png)


---
<style scoped>
{
  font-size: 32px
}
</style>
#### Scause register
When an exception occurs, an event number indicating **causing an interrupt/abnormal event** is written in the CSR, recorded in the ``Exception Code`` field; if the event is caused by an interrupt, the ``Interrupt`` bit is set.
cause register

![w:1150](figs/rv-cause.png)


---
<style scoped>
{
  font-size: 32px
}
</style>
#### Mtvec & stvec registers
The interrupt/exception vector (trap-vector) base address register stvec CSR is used to configure the **trap_handler address**
  - Includes vector base address (BASE) and vector mode (MODE): the value in the BASE field is 4-byte aligned
     - MODE = 0 means a trap_handler handles all interrupts/exceptions
     - MODE = 1 means each interrupt/exception has a corresponding trap_handler

mtvec & stvec registers
![w:1000](figs/rv-tvec.png)



---
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming
   - Interrupt/exception mechanism
   - **Interrupt/Exception Handling**
   - Virtual memory mechanism
---
<style scoped>
{
  font-size: 32px
}
</style>
#### S-Mode interrupt/abnormal hardware response

**Hardware Execution Content**

The hart accepts the interrupt/exception and needs to be delegated to S-Mode, then the hardware will undergo the following state transitions atomically

1. **Interrupt/abnormal instruction PC** is stored in sepc, and PC is set to stvec
2. scause sets interrupt/abnormal **type**, stval is set to error address/abnormal **related information**
3. Set the SIE bit in sstatus to zero, **mask the interrupt**, the value before the **SIE bit** is saved in the SPIE bit


---
#### S-Mode interrupt/abnormal hardware response

4. **The privilege mode before the exception occurred** is saved in the SPP (previous privilege) field of sstatus, and then set the current privilege mode to S-Mode
5. **Jump** to the address set by stvec CSR to continue execution


---
#### S-Mode Interrupt/Exception Software Handling


- **Initialization**
   - Write interrupt/exception handling routines (such as trap_handler)
   - Set trap_handler address to stvec
- Software execution
   1. The processor jumps to **trap_handler**
   2. trap_handler **handling** interrupt/exception/system call, etc.
   3. **Return** to the previous instruction and the previous privilege level to continue execution


---
**Outline**

1. Mainstream CPU Comparison
2. RISC-V system mode
3. RISC-V system programming: user mode programming
4. RISC-V system programming: M-Mode programming
5. RISC-V system programming: kernel programming
   - Interrupt/exception mechanism
   - Interrupt/exception handling
   - **Virtual storage mechanism**
---
<style scoped>
{
  font-size: 27px
}
</style>
#### S-Mode virtual memory system

- Virtual addresses divide memory into **fixed-size pages** for **address translation** and **content protection**.
- satp (Supervisor Address Translation and Protection, supervisor address translation and protection) S mode control status register **control paging**. satp has three domains:
   - The MODE field can **turn on pagination** and select the number of page table levels
   - ASID (Address Space Identifier, Address Space Identifier) field is optional, it can be used to reduce the overhead of context switching
   - The PPN field holds the physical page number of the **root page table**
![w:900](figs/satp.png)

---
#### S-Mode Virtual Storage Mechanism

- Establish **page table base address** through stap CSR
- Create **page tables** for OS and APP
- Handle memory access **Exception**


![bg right:50% 100%](figs/riscv_pagetable.svg)

---
<style scoped>
{
  font-size: 32px
}
</style>
#### S-Mode virtual memory address translation
In S, U-Mode, the virtual address will be converted to a physical address by traversing the page table from the root:

- satp.PPN gives the **first-level page table base address**, VA [31:22] gives the first-level page number, and the CPU will read the address located at (satp.PPN × 4096 + VA[31: 22 ] × 4) Page table entry.
- PTE contains **secondary page table base address**, VA[21:12] gives the second-level page number, CPU reads at address (PTE. PPN × 4096 + VA[21: 12] × 4) leaf Node page table entry.
- **The PPN field of the leaf node page table entry** and the page offset (the lowest 12 effective bits of the original virtual address) form the final result: physical address (LeafPTE.PPN×4096+VA[11: 0])


---
#### S-Mode virtual memory address translation

![w:650](figs/satp2.png)


<!-- ---
## RISC-V System Programming: Isolation in S-Mode
- S-Mode is more privileged than U-Mode, but less privileged than M-Mode
- Software running in S-Mode cannot use M-Mode CSR and instructions, and is restricted by PMP
- Support page-based virtual memory -->




---
### Summary

- Understand RISC-V privilege levels and hardware isolation methods
- Understand the basic characteristics of RISC-V's M-Mode and S-Mode
- Understand how the OS accesses and controls the computer system in M-Mode and S-Mode
- Learn how different software switches between M-Mode<–>S-Mode<–>U-Mode