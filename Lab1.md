# JOS-operating-system
 
## Part 1:
 
Getting familiarized with x86 assembly language, the QEMU x86 emulator, and the PC's power-on bootstrap procedure. 
 

### Exercise 3
 
>At what point does the processor start executing 32-bit code? What exactly causes the switch from 16- to 32-bit mode?

lgdt gdtdesc set up the Global Descriptor Table and cr0 is modified to turn on the protected mode.

>What is the last instruction of the boot loader executed, and what is the first instruction of the kernel it just loaded?

After loading the kernel image, the boot loader finishes as calling ((void (\*)void)) (ELFHDR->e_entry))(), call \*0x10018. The first instruction of the kernel is 0x10000c: $0x1234, 0x472.

>Where is the first instruction of the kernel?

The code is in /kern/entry.S. The instruction is in 0x10000c.

>How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?

It finds out this information from kernel's ELF image. After loading ELF image to 0x10000 as ELFHDR, it finds program header table from ELFHDR + ELFHDR->e_phoff to ELFHDR + ELFHDR->e_phoff + ELFHDR->e_phnum. For each segment ph, it then reads data to the correspoding physical address ph.p_pa.

## Part 2:
 
Examines the boot loader for our 6.828 kernel, which resides in the boot directory of the lab tree. 

### Exercise 5: 
>Trace through the first few instructions of the boot loader again and identify the first instruction that would "break" or otherwise do the wrong thing if you were to get the boot loader's link address wrong. Then change the link address in boot/Makefrag to something wrong, run make clean, recompile the lab with make, and trace into the boot loader again to see what happens. Don't forget to change the link address back and make clean again afterward!

Change the link address to 0x8C00 in boot/Makefrag. Start again from 0x7c00, it went wrong at lgdt instruction: lgdtw -0x739c and got stuck at 0x7c2d: ljmp $0x8, $0x8c32.

### Exercise 6:
>Reset the machine (exit QEMU/GDB and start them again). Examine the 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader, and then again at the point the boot loader enters the kernel. Why are they different? What is there at the second breakpoint? (You do not really need to use QEMU to answer this question. Just think.)

The 8 words of memory at 0x00100000 at the point the BIOS enters the boot loader is 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000 0x00000000.

The 8 words of memory at 0x00100000 at the point the boot loader enters the kernel is 0x1badb002 0x00000000, 0xe4524ffe, 0x7205c766, 0x34000004, 0x0000b812, 0x220f0011, 0xc0200fd8. Compare to the content in obj/kern/kernal.asm, 

```{r}
.globl entry
entry:
	movw	$0x1234,0x472			# warm boot
f0100000:	02 b0 ad 1b 00 00    	add    0x1bad(%eax),%dh
f0100006:	00 00                	add    %al,(%eax)
f0100008:	fe 4f 52             	decb   0x52(%edi)
f010000b:	e4                   	.byte 0xe4

f010000c <entry>:
f010000c:	66 c7 05 72 04 00 00 	movw   $0x1234,0x472
f0100013:	34 12 
	movl	$(RELOC(entry_pgdir)), %eax
f0100015:	b8 00 50 11 00       	mov    $0x115000,%eax
	movl	%eax, %cr3
f010001a:	0f 22 d8             	mov    %eax,%cr3
	# Turn on paging.
	movl	%cr0, %eax
f010001d:	0f 20 c0             	mov    %cr0,%eax
```

it is the corresponding machine code for the boot loader entering the kernel.

## Part 3:
We will now start to examine the minimal JOS kernel in a bit more detail.

### Exercise 7. 
>Use QEMU and GDB to trace into the JOS kernel and stop at the movl %eax, %cr0. Examine memory at 0x00100000 and at 0xf0100000. Now, single step over that instruction using the stepi GDB command. Again, examine memory at 0x00100000 and at 0xf0100000. Make sure you understand what just happened.

>What is the first instruction after the new mapping is established that would fail to work properly if the mapping weren't in place? Comment out the movl %eax, %cr0 in kern/entry.S, trace into it, and see if you were right.

A: movl %eax, %cr0 enables paging. After this instruction, memory at 0x00100000 and at 0xf0100000 becomes the same. The first failed instruction if the mapping weren't in place is jmp *%eax. This is because %Eax stores 0xf010002c, an address outside of RAM.
