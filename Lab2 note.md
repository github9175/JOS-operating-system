# JOS-operating-system

## Lab 2 notes: Memory Management

Part 1: Physical Page Management

Exercise 1

- kern/pmap.c
  - Fields: npages(amount of physical memory in pages), *kern_pgdir(kernel's initial page), *pages(Physical page), * page_free_list(free list of physical page)
  - mem_init(): boot_alloc(), memset(), page_init(), checkXXX use page_alloc() and page_free() 

