# JOS-operating-system

Lab5
The goal for this lab is not to have you implement the entire file system, but for you to implement only certain key components. In particular, you will be responsible for reading blocks into the block cache and flushing them back to disk; allocating disk blocks; mapping file offsets to disk blocks; and implementing read, write, and open in the IPC interface. 

Exercise 2:

Add page fault handler:

A page fault (sometimes called #PF, PF or hard fault[a]) is a type of exception raised by computer hardware when a running program accesses a memory page that is not currently mapped by the memory management unit (MMU) into the virtual address space of a process.

first round addr to page boundary, fs/ide.c has code to read the disk(ide_read);
sys_page_alloc?
sys_page_map?

flush block:
The flush_block function should write a block out to disk if necessary. 

va_is_dirty: We will use the VM hardware to keep track of whether a disk block has been modified since it was last read from or written to disk. To see whether a block needs writing, we can just look to see if the PTE_D "dirty" bit is set in the uvpt entry. (The PTE_D bit is set by the processor in response to a write to that page; see 5.2.4.3 in chapter 5 of the 386 reference manual.)

sys_page_map: After writing the block to disk, flush_block should clear the PTE_D bit using sys_page_map.
PTE_SYSCALL?
 
fs_init function in fs/fs.c is a prime example of how to use the block cache. 

Exercise 3:
alloc_block:  find a free disk block in the bitmap, mark it used, and return the number of that block. When you allocate a block, you should immediately flush the changed bitmap block to disk with flush_block, to help file system consistency.

call: flush_block?

data structure: bitmap. blockno

Exercise 4:
file_block_walk: find disk block number for file block number: alloc_block

file_get_block: call file_block_walk, alloc_block, set *blk

Exercise 5:
serve_read():
call: file_read/fs.c
being called: serve/serv.c

structure: Fsipc

Exercise 6:
serve_write():

devfile_write():
call: fsipc in lib/file.c, memmove
being called: ?
