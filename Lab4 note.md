# JOS-operating-system

Lab4

In this lab you will implement preemptive multitasking among multiple simultaneously active user-mode environments.

In part A you will add multiprocessor support to JOS, implement round-robin scheduling, and add basic environment management system calls (calls that create and destroy environments, and allocate/map memory).

In part B, you will implement a Unix-like fork(), which allows a user-mode environment to create copies of itself.

Finally, in part C you will add support for inter-process communication (IPC), allowing different user-mode environments to communicate and synchronize with each other explicitly. You will also add support for hardware clock interrupts and preemption.

We are going to make JOS support "symmetric multiprocessing" (SMP), a multiprocessor model in which all CPUs have equivalent access to system resources such as memory and I/O buses. While all CPUs are functionally identical in SMP, during the boot process they can be classified into two types: the bootstrap processor (BSP) is responsible for initializing the system and for booting the operating system; and the application processors (APs) are activated by the BSP only after the operating system is up and running. Which processor is the BSP is determined by the hardware and the BIOS. Up to this point, all your existing JOS code has been running on the BSP.

In an SMP system, each CPU has an accompanying local APIC (LAPIC) unit. The LAPIC units are responsible for delivering interrupts throughout the system. The LAPIC also provides its connected CPU with a unique identifier. In this lab, we make use of the following basic functionality of the LAPIC unit (in kern/lapic.c):

Then modify the code in env_alloc() in kern/env.c to ensure that user environments are always run with interrupts enabled.

boot_aps() and mp_main() in kern/init.c

kern/mpentry.S

When running a process, a CPU executes the normal processor loop: read an instruction,
advance the program counter, execute the instruction, repeat. But there are
events on which control from a user program must transferred back to the kernel instead
of executing the next instruction. These events include a device signaling that it
wants attention, a user program doing something illegal (e.g., references a virtual address
for which there is no PTE), or a user program asking the kernel for a service
with a system call. There are three main challenges in handling these events: 1) the
kernel must arrange that a processor switches from user mode to kernel mode (and
back); 2) the kernel and devices must coordinate their parallel activities; and 3) the
kernel must understand the interface of the devices well. Addressing these 3 challenges
requires detailed understanding of hardware and careful programming, and can
result in opaque kernel code.

Any operating system is likely to run with more processes than the computer has
processors, and so a plan is needed to time-share the processors among the processes.
Ideally the sharing would be transparent to user processes. A common approach is to
provide each process with the illusion that it has its own virtual processor, and have
the operating system multiplex multiple virtual processors on a single physical processor.
This chapter explains how xv6 multiplexes a processor among several processes.
