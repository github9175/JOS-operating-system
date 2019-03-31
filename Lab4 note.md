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

As shown in Figure 5-1, to switch between processes, xv6 performs two kinds of
context switches at a low level: from a process’s kernel thread to the current CPU’s
scheduler thread, and from the scheduler thread to a process’s kernel thread. xv6 never
directly switches from one user-space process to another; this happens by way of a
user-kernel transition (system call or interrupt), a context switch to the scheduler, a
context switch to a new process’s kernel thread, and a trap return. In this section we’ll
example the mechanics of switching between a kernel thread and a scheduler thread.
Every xv6 process has its own kernel stack and register set, as we saw in Chapter
2. Each CPU has a separate scheduler thread for use when it is executing the scheduler
rather than any process’s kernel thread. Switching from one thread to another involves
saving the old thread’s CPU registers, and restoring previously-saved registers of
the new thread; the fact that %esp and %eip are saved and restored means that the
CPU will switch stacks and switch what code it is executing.
swtch doesn’t directly know about threads; it just saves and restores register sets,
called contexts. When it is time for the process to give up the CPU, the process’s kernel
thread will call swtch to save its own context and return to the scheduler context.
Each context is represented by a struct context*, a pointer to a structure stored on
the kernel stack involved. Swtch takes two arguments: struct context **old and
struct context *new. It pushes the current CPU register onto the stack and saves the
stack pointer in *old. Then swtch copies new to %esp, pops previously saved registers,
and returns.

In our example, sched called swtch to switch to cpu->scheduler, the per-CPU
scheduler context. That context had been saved by scheduler’s call to swtch (2728).
When the swtch we have been tracing returns, it returns not to sched but to sched-
uler, and its stack pointer points at the current CPU’s scheduler stack, not initproc’s
kernel stack.

The last section looked at the low-level details of swtch; now let’s take swtch as a
given and examine the conventions involved in switching from process to scheduler
and back to process. A process that wants to give up the CPU must acquire the process
table lock ptable.lock, release any other locks it is holding, update its own state
(proc->state), and then call sched. Yield (2772) follows this convention, as do sleep
and exit, which we will examine later. Sched double-checks those conditions (2757-
2762) and then an implication of those conditions: since a lock is held, the CPU should
be running with interrupts disabled. Finally, sched calls swtch to save the current
context in proc->context and switch to the scheduler context in cpu->scheduler.
Swtch returns on the scheduler’s stack as though scheduler’s swtch had returned
(2728). The scheduler continues the for loop, finds a process to run, switches to it, and
the cycle repeats.

The scheduler loops over the process table looking for a runnable process, one
that has p->state == RUNNABLE. Once it finds a process, it sets the per-CPU current
process variable proc, switches to the process’s page table with switchuvm, marks the
process as RUNNING, and then calls swtch to start running it (2722-2728).

One way to think about the structure of the scheduling code is that it arranges to
enforce a set of invariants about each process, and holds ptable.lock whenever those
invariants are not true. One invariant is that if a process is RUNNING, things must be
set up so that a timer interrupt’s yield can correctly switch away from the process;
this means that the CPU registers must hold the process’s register values (i.e. they
aren’t actually in a context), %cr3 must refer to the process’s pagetable, %esp must refer
to the process’s kernel stack so that swtch can push registers correctly, and proc
must refer to the process’s proc[] slot. Another invariant is that if a process is
RUNNABLE, things must be set up so that an idle CPU’s scheduler can run it; this
means that p->context must hold the process’s kernel thread variables, that no CPU
is executing on the process’s kernel stack, that no CPU’s %cr3 refers to the process’s
page table, and that no CPU’s proc refers to the process.

Scheduling and locks help conceal the existence of one process from another, but
so far we have no abstractions that help processes intentionally interact. Sleep and
wakeup fill that void, allowing one process to sleep waiting for an event and another
process to wake it up once the event has happened. Sleep and wakeup are often called
sequence coordination or conditional synchronization mechanisms, and there
are many other similar mechanisms in the operating systems literature.
