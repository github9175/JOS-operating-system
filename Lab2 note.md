# JOS-operating-system

## Lab 2 notes: Memory Management

Physical Page Management： The first component is a physical memory allocator for the kernel, so that the kernel can allocate memory and later free it. Your allocator will operate in units of 4096 bytes, called pages. Your task will be to maintain data structures that record which physical pages are free and which are allocated, and how many processes are sharing each allocated page.

Virtual Memory：The second component of memory management is virtual memory, which maps the virtual addresses used by kernel and user software to addresses in physical memory. The x86 hardware's memory management unit (MMU) performs the mapping when instructions use memory, consulting a set of page tables. You will modify JOS to set up the MMU's page tables according to a specification we provide.

You'll write the physical page allocator. It keeps track of which pages are free with a linked list of struct PageInfo objects (which, unlike xv6, are not embedded in the free pages themselves), each corresponding to a physical page.

- kern/pmap.c
  - mem_init(): 
  - boot_alloc():
  - page_init():
  - page_alloc():
  - page_free() :

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
  
Important macro:
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
    - KERNBASE: 0xF000000
    - KSTACKTOP: KERNBASE 
    - KSTKSIZE: 8*PGSIZE
    - UPAGES: read-only copies of the page structures

  mmu.h: (memory management unit)
   
    A linear address 'la' has a three-part structure: (page directory [PDX] (10 bit) + page table index [PTX] (10 bit)) [PGNUM] + offset within page [PGOFF] (12 bit)
  
    - PGSIZE: 4096 (bytes mapped by a page)
    - PTSIZE: PGSIZE*NPTENTRIES(1024) (bytes mapped by a page directory entry)
    - PTE_P: 0x001 present
    - PTE_W: 0x002 writeable
    - PTE_U: 0x004 user
    - PTE_ADDR: 
