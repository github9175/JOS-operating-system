# JOS-operating-system

## Lab 2: Memory Management

* This lab is about memory management. Memory management has two components. 

  *  The first component is a `physical memeory allocator` for the kernel, so that the kernel can allocate memory and later free it. The allocator will operate in units of 4096 bytes, called pages. The task will be to maintain data structures that record which physical pages are free and which are allocated, and how many processes are sharing each allocated page. You will also write the routines to allocate and free pages of memory.

  *  The second component of memory management is `virtual memory`, which maps the virtual addresses used by kernel and user software to addresses in physical memory. The x86 hardware's memory management unit (MMU) performs the mapping when instructions use memory, consulting a set of page tables. You will modify JOS to set up the MMU's page tables according to a specification we provide.

* Lab 2 contains the following new source files, which you should browse through:

  inc/memlayout.h
  
  kern/pmap.c
  
  kern/pmap.h
  
  kern/kclock.h
  
  kern/kclock.c

  *  *memlayout.h* describes the layout of the virtual address space that you must implement by modifying pmap.c. 

  *  *memlayout.h* and pmap.h define the PageInfo structure that you'll use to keep track of which pages of physical memory are free. 

  *  *kclock.c* and *kclock.h* manipulate the PC's battery-backed clock and CMOS RAM hardware, in which the BIOS records the amount of physical memory the PC contains, among other things. The code in pmap.c needs to read this device hardware in order to figure out how much physical memory there is, but that part of the code is done for you: you do not need to know the details of how the CMOS hardware works.

### Part 1: Physical Page Management

The operating system must keep track of which parts of physical RAM are free and which are currently in use. JOS manages the PC's physical memory with page granularity so that it can use the MMU to map and protect each piece of allocated memory.

You'll now write the physical page allocator. It keeps track of which pages are free with a linked list of struct PageInfo objects (which, unlike xv6, are not embedded in the free pages themselves), each corresponding to a physical page.

#### Exercise 1.

>In the file kern/pmap.c, you must implement code for the following functions (probably in the order given). boot_alloc(), mem_init() (only up to the call to check_page_free_list(1)), page_init(), page_alloc(), page_free(). check_page_free_list() and check_page_alloc() test your physical page allocator. You should boot JOS and see whether check_page_alloc() reports success. Fix your code so that it passes. You may find it helpful to add your own assert()s to verify that your assumptions are correct.
