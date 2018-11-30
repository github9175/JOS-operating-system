# JOS-operating-system

## Lab 2 notes: Memory Management

Part 1: Physical Page Management

Exercise 1

- kern/pmap.c
  - boot_alloc(n): used in kern_pgdir = (pde_t *) boot_alloc(PGSIZE); allocates enough pages of contiguous physical memory to hold n bytes and return the address;
  - mem_init()

