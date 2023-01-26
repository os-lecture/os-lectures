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

# Lecture 2 Introduction to Practice and Experiments
## Section 4 Practice: Bare Metal Program -- LibOS

<br>
<br>

Xiang Yong Chen Yu Li Guoliang

<br>
<br>

Fall 2022

---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

### 1. Experimental objectives and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
7. Function calls
8. LibOS initialization
9. SBI calls

---
<style scoped>
{
  font-size: 28px
}
</style>
#### Experimental goals for LibOS

Bare Metal Program (Bare Metal Program): an OS-type program that has nothing to do with the operating system

- Establish the execution environment of the application
   - Keep applications isolated from hardware
   - Simplifies the difficulty and complexity of applications accessing hardware
- **Execution Environment (Execution Environment)**: Responsible for providing corresponding functions and resources to the software executed on it. The multi-level **software and hardware system**
 
![bg right:30% 100%](figs/os-as-lib.png)


---
<style scoped>
{
  font-size: 32px
}
</style>

#### LibOS Historical Background
From 1949 to 1951, the British J. Lyons and Co. (a chain restaurant and food manufacturing group company) pioneered the introduction and use of the EDSAC computer of the University of Cambridge, and jointly designed and implemented the LEO I 'Lyons Electronic Office' hardware and software system


![bg right:50% 70%](figs/LEO_III_computer_circuit_board.jpg)



---
<style scoped>
{
  font-size: 28px
}
</style>

#### LibOS Historical Background -- Subroutines

- David Wheeler, who worked on the EDSAC project, invented the concept of **subroutines** – Wheeler Jump
- With the convenient and effective subroutine concept and subroutine calling mechanism, software developers developed a large number of system **subroutine libraries** on EDSAC and subsequent LEO computers, forming the earliest operating system prototype.

![bg right:50% 70%](figs/LEO_III_computer_circuit_board.jpg)


---

#### General idea of LibOS
- Compilation: Support compiling bare-metal programs by setting the compiler
- Construction: build the stack of the bare metal program and the SBI service request interface
- Run: OS start address and execution environment initialization

![bg right:45% 100%](figs/os-as-lib.png)



---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
### 2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
7. Function calls
8. LibOS initialization
9. SBI calls

---
#### Understand the execution process of LibOS
- Can write/compile/run bare metal programs
- Understand function calls based on bare metal programs
- Can understand assembly code pseudo code
- Can understand inline assembly code
- Preliminary understanding of SBI calls
![bg right:35% 100%](figs/os-as-lib.png)
---
#### Master the basic concepts
- **Can write the trilobite operating system now!**
   - What is an ABI?
   - What is SBI?
   - Supervisor Binary Interface?
![bg right:50% 90%](figs/trilobita.png)

Note: Trilobite Trilobita is the most representative ancient animal in the Cambrian Period

---
#### Analyze execution details

- **Understanding functions at the machine level**
   - registers
   - function call/return (call/return)
   - Function enter/exit (enter/exit)
   - Function prologue/epilogue (prologue/epilogue)

![bg right:50% 90%](figs/trilobita.png)

---

#### The OS is not always the bottom layer of the software
   - There is a sky
   -Why?

![bg right:50% 90%](figs/trilobita.png)

---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
### 3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
7. Function calls
8. LibOS initialization
9. SBI calls

---

#### Practical steps
- Build a development and experiment environment
- Remove standard library dependencies
- Support function call
- Complete output and shutdown based on SBI service

**Understand the memory space and stack of running programs**

![bg right:45% 100%](figs/os-as-lib.png)

---

#### Steps
```
git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
cd rCore-Tutorial-v3
git checkout ch1

cd os
make run
```

---

#### Results of the
```
[RustSBI output]
Hello, world!
.text [0x80200000, 0x80202000)
.rodata [0x80202000, 0x80203000)
.data [0x80203000, 0x80203000)
boot_stack [0x80203000, 0x80213000)
.bss [0x80213000, 0x80213000)
Panicked at src/main.rs:46 Shutdown machine!
```

In addition to displaying Hello, world! There are some additional information, and finally shut down.

---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
### 4. Code structure
1. Memory layout
2. Verify the startup process based on GDB
3. Function calls
4. LibOS initialization
5. SBI calls

![bg right:50% 100%](figs/lib-os-detail.png)

---
#### LibOS code structure
```
./os/src
Rust 4 Files 119 Lines
Assembly 1 Files 11 Lines

├── bootloader (the implementation of SBI running at the M privilege level that the kernel depends on, we use RustSBI in this project)
│ ├── rustsbi-k210.bin (precompiled binary version that can run on k210 real hardware platform)
│ └── rustsbi-qemu.bin (precompiled binary version that can run on the qemu virtual machine)
```

---
#### LibOS code structure
```
├── os (our kernel implementation is placed in the os directory)
│ ├── Cargo.toml (some configuration files implemented by the kernel)
│ ├── Makefile
│ └── src (all kernel source codes are placed in the os/src directory)
│ ├── console.rs (to further encapsulate the SBI interface of printing characters to achieve more powerful formatted output)
│ ├── entry.asm (a piece of assembly code to set the kernel execution environment)
│ ├── lang_items.rs (some semantic items we need to provide to the Rust compiler, currently contains the processing logic when the kernel panics)
│ ├── linker-qemu.ld (the linker script that controls the kernel memory layout to make the kernel run on the qemu virtual machine)
│ ├── main.rs (kernel main function)
│ └── sbi.rs (call the SBI interface provided by the underlying SBI implementation)
```


<!-- https://blog.51cto.com/onebig/2551726
(In-depth understanding of computer systems) bss segment, data segment, text segment, heap (heap) and stack (stack) -->


---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
### 5. Memory layout
1. Verify the startup process based on GDB
2. Function calls
3. LibOS initialization
4. SBI calls

---
#### App/OS Memory Layout
![bg w:950](figs/memlayout.png)

---
#### Sss section

- The bss segment (bss segment) usually refers to a memory area used to store **uninitialized global variables** in the program
- bss is the abbreviation of English Block Started by Symbol
- The bss segment belongs to **static memory allocation**
![bg right:45% 130%](figs/memlayout.png)


---
#### Data section

- The data segment (data segment) usually refers to a memory area used to store **initialized global variables** in the program
- The data segment belongs to **static memory allocation**
![bg right:50% 130%](figs/memlayout.png)



---
#### Text section

- Code segment (code segment/text segment) refers to the memory area that stores **executive code**
- The size of this part of the area is determined, usually **read-only**
- In the code segment, it is also possible to contain some **read-only constant variables**
![bg right:45% 130%](figs/memlayout.png)



---
#### Heap

- The heap is a memory segment used for **dynamic allocation**, which can be dynamically expanded or reduced
- The program calls **malloc** and other functions to allocate newly allocated memory to be dynamically added to the heap
- The memory freed by calling functions such as **free** is removed from the heap
![bg right:45% 130%](figs/memlayout.png)


---
<style scoped>
{
  font-size: 30px
}
</style>

#### Stack

- The stack, also known as the stack, is a **local variable** temporarily created by the user to save the program
- When a function is called, its **parameters** and the **return value** of the function will also be placed on the stack
- Due to the **first-in last-out** feature of the stack, the stack is particularly convenient for saving/restoring the current execution state



![bg right:45% 130%](figs/memlayout.png)


---
#### Stack

The stack can be regarded as a memory area for **registering and exchanging temporary data**

A significant difference between OS programming and application programming is that OS programming requires an understanding of the **physical memory structure on the stack** and the **machine-level content**.

![bg right:45% 130%](figs/memlayout.png)



---

#### Link-time memory layout customization
```
# os/src/linker-qemu.ld
OUTPUT_ARCH(riscv)
ENTRY(_start)
BASE_ADDRESS = 0x80200000;

SECTIONS
{
     . = BASE_ADDRESS;
     skernel = .;

     text = .;
     .text : {
       *(.text.entry)
```
![bg right:45% 130%](figs/memlayout.png)

---
#### Link-time memory layout customization
```
     .bss : {
         *(.bss.stack)
         sbss = .;
         *(.bss.bss.*)
         *(.sbss.sbss.*)
```

BSS: Block Started by Symbol
SBSS: small bss, near data


![bg right:45% 130%](figs/memlayout.png)



---

#### Generate kernel binary image

![bg w:850](figs/load-into-qemu.png)

---

#### Generate kernel binary image

```
rust-objcopy --strip-all \
target/riscv64gc-unknown-none-elf/release/os \
-O binary target/riscv64gc-unknown-none-elf/release/os.bin
```

---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
### 6. Verify startup process based on GDB
7. Function calls
8. LibOS initialization
9. SBI calls

---

#### Verify startup process based on GDB
```
qemu-system-riscv64 \
     -machine virt \
     -nographic \
     -bios ../bootloader/rustsbi-qemu.bin\
     -device loader,file=target/riscv64gc-unknown-none-elf/release/os.bin,addr=0x80200000 \
     -s -S
```

```
riscv64-unknown-elf-gdb \
     -ex 'file target/riscv64gc-unknown-none-elf/release/os' \
     -ex 'set arch riscv:rv64' \
     -ex 'target remote localhost:1234'
[GDB output]
0x0000000000001000 in ?? ()
```


---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
### 7. Function call
8. LibOS initialization
9. SBI calls

---
#### Compilation principle support for function calls

* Compilation principle course -- [implement function call compilation support](https://decaf-lang.github.io/minidecaf-tutorial/docs/step9/example.html)
* [Documentation for Quick Start RISC-V Assembly](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md)

![bg right:60% 90%](figs/function-call.png)


---
<style scoped>
{
  font-size: 32px
}
</style>
#### call/return directive

Directives | Basic instructions | Meaning |
:----------------|:-----------|:----------|
ret | jalr x0, x1, 0 | function returns
call offset | auipc x6, offset[31:12]; jalr x1, x6, offset[11:0] | function call

auipc (add upper immediate to pc) is used to construct a PC-relative address, using a U-type immediate value. auipc fills the lower 12 bits with 0, and the upper 20 bits are U-type immediate data to form a 32-bit offset, then add it to the PC, and finally save the result in the register x1.

---
<style scoped>
{
  font-size: 32px
}
</style>
#### Function call jump instruction
![w:1000](figs/fun-call-in-rv.png)

The pseudo-instruction ret is translated as jalr x0, 0(x1), which means jumping to the address saved by the register ra (namely x1).
*[Documentation for Quick Start RISC-V Assembly](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md)*


---
<style scoped>
{
  font-size: 32px
}
</style>
#### call/return directive
Directives | Basic instructions | Meaning |
:----------------|:-----------|:----------|
ret | jalr x0, x1, 0 | function return
call offset | auipc x6, offset[31:12]; jalr x1, x6, offset[11:0] | function call

Function call core mechanism
- When the function is called, save the return address and realize the jump through the call pseudo-instruction;
- When the function returns, return to the next instruction before the jump through the ret pseudo-instruction to continue execution


---
#### Function calling convention

The function calling convention (Calling Convention) stipulates how to implement the function call of a certain programming language on a certain instruction set architecture. It includes the following:

- How to pass the input parameters and return value of the function;
- The division of caller/callee saved registers in the function call context;
- Other methods of using registers in the function call process.

---

#### RISC-V function calling convention: call parameters and return value transfer
![w:1200](figs/rv-call-regs.png)
- RISC-V32: If the return value is 64bit, use a0~a1 to place it
- RISC-V64: If the return value is 64bit, use a0 to place it


---

#### RISC-V function calling convention: stack frame
![w:1000](figs/call-stack.png)

---
#### RISC-V function calling convention: stack frame

**Stack Frames**
```
return address *
previous fp
saved registers
local variables
…
return address fp register
previous fp (pointed to *)
saved registers
local variables
… sp register
```
![bg right:50% 180%](figs/stack-frame.png)



---
<style scoped>
{
  font-size: 32px
}
</style>
#### RISC-V function calling convention: stack frame
- Stack frames may have different sizes and contents, but the overall structure is similar
- Each stack frame starts with the **return value** of this function and the fp value of the **previous function**
- The sp register always points to the **bottom** of the current stack frame
- The fp register always points to the **top** of the current stack frame

![bg right:40% 180%](figs/stack-frame.png)



---

#### RISC-V function calling convention: ret instruction
- When the ret instruction is executed, the following pseudo-code realizes adjusting the stack pointer and PC:
```
pc = return address
sp = fp + ENTRY_SIZE
fp = previous fp
```
![bg right:50% 180%](figs/stack-frame.png)

---

#### RISC-V function calling convention: function structure
Function structure: ``prologue``, ``body part`` and ``epilogue``
```
.global sum_then_double
sum_then_double:
addi sp, sp, -16 # prologue
sd ra, 0(sp)

call sum_to # body part
li t0, 2
mul a0, a0, t0

ld ra, 0(sp) # epilogue
addi sp, sp, 16
ret
```

---
#### RISC-V function calling convention: function structure

Function structure: ``prologue``, ``body part`` and ``epilogue``
```
.global sum_then_double
sum_then_double:

call sum_to # body part
li t0, 2
mul a0, a0, t0

ret
```
Q: How does the execution of the above code compare to the execution of the code on the previous page?

<!-- https://blog.csdn.net/zoomdy/article/details/79354502 RISC-V Assembly Programmer's Manual
https://shakti.org.in/docs/risc-v-asm-manual.pdf
https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md
-->

---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
7. Function calls
### 8. LibOS initialization
9. SBI calls

---
<style scoped>
{
  font-size: 32px
}
</style>
#### Allocate and use the startup stack

Allocate and use the startup stack *[Quickstart RISC-V assembly documentation](https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md)*
```
# os/src/entry.asm
     .section.text.entry
     .globl_start
_start:
     la sp, boot_stack_top
     call rust_main

     .section .bss.stack
     .globl boot_stack
boot_stack:
     .space 4096 * 16
     .globl boot_stack_top
boot_stack_top:
```

---
#### Allocate and use the startup stack
```
# os/src/linker-qemu.ld
.bss : {
     *(.bss.stack)
     sbss = .;
     *(.bss.bss.*)
     *(.sbss.sbss.*)
}
ebss = .;
```
In the linker script linker.ld the .bss.stack section will eventually be pooled into the .bss section
The .bss section generally places data that needs to be initialized to zero


---
#### Transfer of Control: ASM --> Rust/C

Transfer control to Rust code, the entry point is the ``rust_main`` function in main.rs
```rust
// os/src/main.rs
pub fn rust_main() -> ! {
     loop {}
}
```
- ``fn`` keyword: function; ``pub`` keyword: externally visible, public
- ``loop`` keyword: loop



---
#### Clear the bss segment

Clear the bss segment (uninitialized data segment)
```Rust
pub fn rust_main() -> ! {
     clear_bss(); //call clear bss function clear_bss()
}
fn clear_bss() {
     extern "C" {
         fn sbss(); //Start address of bss segment
         fn ebss(); //end address of bss segment
     }
     //Clear the memory space of [sbss..ebss]
     (sbss as usize..ebss as usize).for_each(|a| {
         unsafe { (a as *mut u8).write_volatile(0) }
     });
}
```


---
<style scoped>
{
  font-size: 32px
}
</style>
Outline

1. Experimental goals and ideas
2. Experimental requirements
3. Practical steps
4. Code structure
5. Memory layout
6. Verify the startup process based on GDB
7. Function calls
8. LibOS initialization
### 9. SBI call

---
<style scoped>
{
  font-size: 30px
}
</style>
#### SBI service interface
Print Hello world! to the screen
* SBI service interface
     - Supervisor Binary Interface
     - Services provided by lower-level software to the operating system
* RustSBI
     - Implement basic SBI services
     - follow the SBI calling convention
![bg right:50% 100%](figs/rv-privil-arch.png)
---

#### SBI Service Number
```rust
// os/src/sbi.rs
const SBI_SET_TIMER: usize = 0;
const SBI_CONSOLE_PUTCHAR: usize = 1;
const SBI_CONSOLE_GETCHAR: usize = 2;
const SBI_CLEAR_IPI: usize = 3;
const SBI_SEND_IPI: usize = 4;
const SBI_REMOTE_FENCE_I: usize = 5;
const SBI_REMOTE_SFENCE_VMA: usize = 6;
const SBI_REMOTE_SFENCE_VMA_ASID: usize = 7;
const SBI_SHUTDOWN: usize = 8;
```
- ``usize`` machine word-sized unsigned int
---
#### Assembly-level SBI calls

```rust
// os/src/sbi.rs
#[inline(always)] //Always expand the function
fn sbi_call(which: usize, arg0: usize, arg1: usize, arg2: usize) -> usize {
     let mut ret; // modifiable variable ret
     unsafe {
         asm!(//Inline assembly
             "ecall", //Switch to a higher privilege level machine instruction
             inlateout("x10") arg0 => ret, //SBI parameter 0& return value
             in("x11") arg1, //SBI parameter 1
             in("x12") arg2, //SBI parameter 2
             in("x17") which, //SBI number
         );
     }
     ret // return ret value
}
```
---
#### SBI calls: output characters

print a character on the screen
```rust
// os/src/sbi.rs
pub fn console_putchar(c: usize) {
     sbi_call(SBI_CONSOLE_PUTCHAR, c, 0, 0);
}
```
Implement formatted output
- Write a console_putchar based println! macro

---
#### SBI call: Shutdown
```rust
// os/src/sbi.rs
pub fn shutdown() -> ! {
     sbi_call(SBI_SHUTDOWN, 0, 0, 0);
     panic!("It should shutdown!");
}
```
- ``panic!`` and ``println!`` are a macro (like a C macro), and ``!`` is a macro flag
---
#### Handle error panic gracefully
```rust
#[panic_handler]
fn panic(info: &PanicInfo) -> ! { //PnaicInfo is a structure type
     if let Some(location) = info.location() { //Does the error location exist?
         println!(
             "Panicked at {}:{} {}",
             location.file(), //error file name
             location.line(), //The number of lines in the error file
             info.message().unwrap() //error message
         );
     } else {
         println!("Panicked: {}", info. message(). unwrap());
     }
     shutdown() //shutdown
}
```

---
#### LibOS Full Features
Handle panics gracefully
```rust
pub fn rust_main() -> ! {
     clear_bss();
     println!("Hello, world!");
     panic!("Shutdown machine!");
}
```
operation result
```
[RustSBI output]
Hello, world!
Panicked at src/main.rs:26 Shutdown machine!
```

---
### Summary
- Knowledge points that need to be mastered in the practice of constructing various OS (principle & implementation)
- Understand the relationship between Compiler/OS/Machine
- Know the process from machine startup to application printing out strings
- Can write Trilobita OS