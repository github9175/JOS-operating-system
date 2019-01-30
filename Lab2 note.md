# JOS-operating-system

## Lab 2 notes: Memory Management

An x86 page table is logically an array of 2^20 (1,048,576) page table entries
(PTEs). Each PTE contains a 20-bit physical page number (PPN) and some flags. The
paging hardware translates a virtual address by using its top 20 bits to index into the
page table to find a PTE, and replacing the address’s top 20 bits with the PPN in the
PTE. The paging hardware copies the low 12 bits unchanged from the virtual to the
translated physical address

The level of indirection provided by page tables allows many neat tricks. xv6
uses page tables primarily to multiplex address spaces and to protect memory.

the actual translation happens in two steps. A page table
is stored in physical memory as a two-level tree. The root of the tree is a 4096-byte
page directory that contains 1024 PTE-like references to page table pages. Each
page table page is an array of 1024 32-bit PTEs. The paging hardware uses the top 10
bits of a virtual address to select a page directory entry. If the page directory entry is
present, the paging hardware uses the next 10 bits of the virtual address to select a
PTE from the page table page that the page directory entry refers to.

To review, xv6 ensures that each process can use only its own memory. And, each
process sees its memory as having contiguous virtual addresses starting at zero, while
the process’s physical memory can be non-contiguous.

Physical Page Management： The first component is a physical memory allocator for the kernel, so that the kernel can allocate memory and later free it. Your allocator will operate in units of 4096 bytes, called pages. Your task will be to maintain data structures that record which physical pages are free and which are allocated, and how many processes are sharing each allocated page.

Virtual Memory：The second component of memory management is virtual memory, which maps the virtual addresses used by kernel and user software to addresses in physical memory. The x86 hardware's memory management unit (MMU) performs the mapping when instructions use memory, consulting a set of page tables. You will modify JOS to set up the MMU's page tables according to a specification we provide.

You'll write the physical page allocator. It keeps track of which pages are free with a linked list of struct PageInfo objects (which, unlike xv6, are not embedded in the free pages themselves), each corresponding to a physical page.

things don’t understand:
memset,
kern_pgdir[PDX(UVPT)]=PADDR(kern_pgdir)|PTE_U|PTE_P

- kern/pmap.c
  - boot_alloc(): allocate memory, returns virtual address?
  - mem_init(): create initial page directory, form a virtual page table, set up in pages an npages array of PageInfo: page_init(). set up virtual memory mapping : boot_map_region()

  - page_init(): initialize pages and page_free_list.
  - page_alloc(): allocates the last free page.
  - page_free() : add a new page to free_page_list.
  - boot_map_region(): map(va, va+size) of virtual address space to physical(pa, pa+size)
  - pgdir_walk(): Given 'pgdir', a pointer to a page directory, pgdir_walk returns a pointer to the page table entry (PTE) for linear address 'va'.
  - page_insert(): map the physical page 'pp' at virtual address 'va'
  - page_lookup(): return the page mapped at virtual address 'va'
  - page_remove(): unmaps the physical page at virtual address 'va'
  
variables:
  npages: amount of physical memory in pages
  npages_basemem: amount of base memory in pages
  kern_pgdir: kernel's initial page directory, PGSIZE, physical to virtual address?
  pages: physical page array
  page_free_list: a linked list of free physical pages
  bootstack:?
 
conversion among kernel virtual address, physical address, and linear address
Important macro:
  types.h
    - size_t: uint32_t (memory object sizes)
    - pde_t: uint32_t
    - uintptr_t: uint32_t (numerical values of virtual addresses)
    - phyaddr_t: uint32_t (physical addresses)
    
  pmap.h:
    - page2kva: KADDR(page2pa(x))
    - page2pa: pageinfo to physaddr_t, (pp-pages) << PGSHIFT
    - KADDR: _kaddr, physical address to kernel virtual address, x+KERNBASE
    - PADDR: _paddr, kernel virtual address to physical address, x-KERNBASE
    
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
    - PTE_ADDR: x&~0xFFF, address in page table or page directory entry
