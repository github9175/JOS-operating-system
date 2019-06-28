# JOS-operating-system

## Introduction
This project is the course project of MIT's 6.828: Operating System Engineering (https://pdos.csail.mit.edu/6.828/2018/overview.html).

The labs are split into 6 major parts that build on each other, culminating in a primitive operating system on which one can run simple commands through his own shell.

The operating system we are building is called JOS, has Unix-like functions (e.g., fork, exec), but is implemented in an exokernel style (i.e., the Unix functions are implemented mostly in a user-level library instead of in the kernel). The major parts of the JOS operating system are:

1.Booting
2.Memory management
3.User environments
4.Preemptive multitasking
5.File system, spawn, and shell
6.Network driver

To simplify development we use a complete machine simulator (QEMU) for development and debugging.

## Deployment
two sets of tools in this class: an x86 emulator, QEMU, for running your kernel; and a compiler toolchain, including assembler, linker, C compiler, and debugger, for compiling and testing your kernel. More information, see https://pdos.csail.mit.edu/6.828/2018/tools.html.

## Developement Logs

Lab1: Booting Finished
Lab2: Memory management Exercise 7.
Lab3: User environments not start
Lab4: Preemptive multitasking not start
Lab5: File system, spawn, and shell not start
Lab6: Network driver not start
