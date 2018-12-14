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
  
  variables:
  npages: amount of physical memory in pages
  npages_basemem: amount of base memory in pages
  kern_pgdir: kernel's initial page directory
  pages: physical page state array
  page_free_list: free list of physical pages
  bootstack:?
  
  macro:
  types.h
    - size_t: uint32_t (memory object sizes)
    - pde_t: uint32_t
    - uintptr_t: uint32_t (numerical values of virtual addresses)
    - phyaddr_t: uint32_t (physical addresses)
    
  pmap.h:
    - page2pa: pageinfo to physaddr_t, (pp-pages) << PGSHIFT
    - KADDR: _kaddr, physical address to kernel virtual address
    - PADDR: _paddr, kernel virtual address to physical address
    
   memlayout.h:
   /*
 * Virtual memory map:                                Permissions
 *                                                    kernel/user
 *
 *    4 Gig -------->  +------------------------------+
 *                     |                              | RW/--
 *                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 *                     :              .               :
 *                     :              .               :
 *                     :              .               :
 *                     |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~| RW/--
 *                     |                              | RW/--
 *                     |   Remapped Physical Memory   | RW/--
 *                     |                              | RW/--
 *    KERNBASE, ---->  +------------------------------+ 0xf0000000      --+
 *    KSTACKTOP        |     CPU0's Kernel Stack      | RW/--  KSTKSIZE   |
 *                     | - - - - - - - - - - - - - - -|                   |
 *                     |      Invalid Memory (*)      | --/--  KSTKGAP    |
 *                     +------------------------------+                   |
 *                     |     CPU1's Kernel Stack      | RW/--  KSTKSIZE   |
 *                     | - - - - - - - - - - - - - - -|                 PTSIZE
 *                     |      Invalid Memory (*)      | --/--  KSTKGAP    |
 *                     +------------------------------+                   |
 *                     :              .               :                   |
 *                     :              .               :                   |
 *    MMIOLIM ------>  +------------------------------+ 0xefc00000      --+
 *                     |       Memory-mapped I/O      | RW/--  PTSIZE
 * ULIM, MMIOBASE -->  +------------------------------+ 0xef800000
 *                     |  Cur. Page Table (User R-)   | R-/R-  PTSIZE
 *    UVPT      ---->  +------------------------------+ 0xef400000
 *                     |          RO PAGES            | R-/R-  PTSIZE
 *    UPAGES    ---->  +------------------------------+ 0xef000000
 *                     |           RO ENVS            | R-/R-  PTSIZE
 * UTOP,UENVS ------>  +------------------------------+ 0xeec00000
 * UXSTACKTOP -/       |     User Exception Stack     | RW/RW  PGSIZE
 *                     +------------------------------+ 0xeebff000
 *                     |       Empty Memory (*)       | --/--  PGSIZE
 *    USTACKTOP  --->  +------------------------------+ 0xeebfe000
 *                     |      Normal User Stack       | RW/RW  PGSIZE
 *                     +------------------------------+ 0xeebfd000
 *                     |                              |
 *                     |                              |
 *                     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 *                     .                              .
 *                     .                              .
 *                     .                              .
 *                     |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|
 *                     |     Program Data & Heap      |
 *    UTEXT -------->  +------------------------------+ 0x00800000
 *    PFTEMP ------->  |       Empty Memory (*)       |        PTSIZE
 *                     |                              |
 *    UTEMP -------->  +------------------------------+ 0x00400000      --+
 *                     |       Empty Memory (*)       |                   |
 *                     | - - - - - - - - - - - - - - -|                   |
 *                     |  User STAB Data (optional)   |                 PTSIZE
 *    USTABDATA ---->  +------------------------------+ 0x00200000        |
 *                     |       Empty Memory (*)       |                   |
 *    0 ------------>  +------------------------------+                 --+
 *
 * (*) Note: The kernel ensures that "Invalid Memory" is *never* mapped.
 *     "Empty Memory" is normally unmapped, but user programs may map pages
 *     there if desired.  JOS user programs map pages temporarily at UTEMP.
 */
    - KERNBASE: 0xF000000
    - KSTACKTOP: KERNBASE 
    - KSTKSIZE: 8*PGSIZE
    - UPAGES: read-only copies of the page structures

   mmu.h (memory management unit)
 
    Paging data structures: A linear address 'la' has a three-part structure as follows:
    +--------10------+-------10-------+---------12----------+
    | Page Directory |   Page Table   | Offset within Page  |
    |      Index     |      Index     |                     |
    +----------------+----------------+---------------------+
    \--- PDX(la) --/ \--- PTX(la) --/ \---- PGOFF(la) ----/
    \---------- PGNUM(la) ----------/
  
    - PGSIZE: 4096 (bytes mapped by a page)
    - PTSIZE: PGSIZE*NPTENTRIES(1024) (bytes mapped by a page directory entry)
    - PTE_P: 0x001 present
    - PTE_W: 0x002 writeable
    - PTE_U: 0x004 user
    - PTE_ADDR: 
