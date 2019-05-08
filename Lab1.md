# JOS-operating-system
 
Part 1:
 
Getting familiarized with x86 assembly language, the QEMU x86 emulator, and the PC's power-on bootstrap procedure. 
 
Exercise 3
 
Q:At what point does the processor start executing 32-bit code? What exactly causes the switch from 16- to 32-bit mode?

A: lgdt gdtdesc set up the Global Descriptor Table and cr0 is modified to turn on the protected mode.

Q:What is the last instruction of the boot loader executed, and what is the first instruction of the kernel it just loaded?

A:After loading the kernel image, the boot loader finishes as calling ((void (*)void)) (ELFHDR->e_entry))(), call *0x10018. The first instruction of the kernel is $0x1234, 0x472.

Q:Where is the first instruction of the kernel?

A: /kern/entry.S.

Q:How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?

A: It finds out this information from kernel's ELF image. After loading ELF image to 0x10000 as ELFHDR, it finds program header table from ELFHDR + ELFHDR->e_phoff to ELFHDR + ELFHDR->e_phoff + ELFHDR->e_phnum. For each segment ph, it then reads data to the correspoding physical address ph.p_pa.

Part 2:
 
Examines the boot loader for our 6.828 kernel, which resides in the boot directory of the lab tree. 

Exercise 6:
Reset the machine (exit QEMU/GDB and start them again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different? What is there at the second breakpoint? (You do not really need to use QEMU to answer this question. Just think.)

A: The 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader is 0x00000000 * 8.
The 8 words of memory at 0x00100000 at the point the boot loader enters the kernel is 0x00000000 * 8.


Part 3:
 
