# JOS-operating-system

## Lab 2 notes: Memory Management

Part 1: Physical Page Management

Exercise 1

- kern/pmap.c
  - Fields: npages(amount of physical memory in pages), *kern_pgdir(kernel's initial page), *pages(Physical page), * page_free_list(free list of physical page)
  - mem_init(): boot_alloc(), memset(), page_init(), checkXXX use page_alloc() and page_free() 

  - pgdir_walk(): Given 'pgdir', a pointer to a page directory, pgdir_walk returns a pointer to the page table entry (PTE) for linear address 'va'.
  - boot_map_region(): map(va, va+size) of virtual address space to physical(pa, pa+size)
  - page_insert(): map the physical page 'pp' at virtual address 'va'
  - page_lookup(): return the page mapped at virtual address 'va'
  - page_remove(): unmaps the physical page at virtual address 'va'
  macro:
    - PageInfo:
    - page2pa:
    - KADDR:
    - PTE_ADDR:
    - PTE_P:
    - PTE_W:
    - PTE_U:
    - PDX:
    - pde_t
    - uintptr_t
    - size_t
    - phyaddr_t
