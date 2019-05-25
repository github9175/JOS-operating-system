# JOS-operating-system

## Lab 2: Memory Management

* This lab is about memory management. It has two components. The allocator will operate in units of 4096 bytes, called pages.

The first component is a physical memeory allocator for the kernel, so that the kernel can allocate memory and later free it.

The task will be to maintain data structures that record which physical pages are free and which are allocated, and how many processes are sharing each allocated page. You will also write the routines to allocate and free pages of memory.

The second component of memory management is virtual memory, which maps the virtual addresses used by kernel and user software to addresses in physical memory. The x86 hardware's memory management unit (MMU) performs the mapping when instructions use memory, consulting a set of page tables. You will modify JOS to set up the MMU's page tables according to a specification we provide.

* Lab 2 contains the following new source files, which you should browse through:

inc/memlayout.h
kern/pmap.c
kern/pmap.h
kern/kclock.h
kern/kclock.c

memlayout.h describes the layout of the virtual address space that you must implement by modifying pmap.c. 

memlayout.h and pmap.h define the PageInfo structure that you'll use to keep track of which pages of physical memory are free. 

kclock.c and kclock.h manipulate the PC's battery-backed clock and CMOS RAM hardware, in which the BIOS records the amount of physical memory the PC contains, among other things. The code in pmap.c needs to read this device hardware in order to figure out how much physical memory there is, but that part of the code is done for you: you do not need to know the details of how the CMOS hardware works.
