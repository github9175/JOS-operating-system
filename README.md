# JOS-operating-system

## Introduction
This project is the course project of MIT's 6.828: Operating System Engineering (https://pdos.csail.mit.edu/6.828/2018/overview.html).

The operating system we are building is called JOS, has Unix-like functions (e.g., fork, exec), but is implemented in an exokernel style (i.e., the Unix functions are implemented mostly in a user-level library instead of in the kernel).

The labs are split into 6 major parts that build on each other, culminating in a primitive operating system on which one can run simple commands through his own shell.
* Booting
* Memory management
* User environments
* Preemptive multitasking
* File system, spawn, and shell
* Network driver

To simplify development we use a complete machine simulator (QEMU) for development and debugging.

## Compilation & Execution

Two sets of tools: 
* an x86 emulator, QEMU, for running the kernel
* a compiler toolchain, including assembler, linker, C compiler, and debugger, for compiling and testing the kernel. 
More information, see https://pdos.csail.mit.edu/6.828/2018/tools.html.

## Development Log
| Lab  | Content | Status | Report | 
| ---- | ---- | ---- | ---- |
| Lab1 | Booting | Finished |[Report1](https://github.com/smharb/JOS-operating-system/blob/master/Lab1%20Report.md)|
| Lab2 | Memory management | Exercise 4 |[Report2](https://github.com/smharb/JOS-operating-system/blob/master/Lab2%20Report.md)|
| Lab3 | User environments | not start ||
| Lab4 | Preemptive multitasking | not start | |
| Lab5 | File system, spawn, and shell | not start | |
| Lab6 | Network driver | not start | |
