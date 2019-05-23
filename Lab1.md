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

It finds out this information from kernel's ELF image. After loading ELF image to 0x10000 as ELFHDR, it finds program header table from ELFHDR + ELFHDR->e_phoff to ELFHDR + ELFHDR->e_phoff + ELFHDR->e_phnum. For each segment ph, it then reads data to the corresponding physical address ph.p_pa.

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
    movw    $0x1234,0x472            # warm boot
f0100000:    02 b0 ad 1b 00 00        add    0x1bad(%eax),%dh
f0100006:    00 00                    add    %al,(%eax)
f0100008:    fe 4f 52                 decb   0x52(%edi)
f010000b:    e4                       .byte 0xe4

f010000c <entry>:
f010000c:    66 c7 05 72 04 00 00     movw   $0x1234,0x472
f0100013:    34 12 
    movl    $(RELOC(entry_pgdir)), %eax
f0100015:    b8 00 50 11 00           mov    $0x115000,%eax
    movl    %eax, %cr3
f010001a:    0f 22 d8                 mov    %eax,%cr3
    # Turn on paging.
    movl    %cr0, %eax
f010001d:    0f 20 c0                 mov    %cr0,%eax
```

it is the corresponding machine code for the boot loader to enter the kernel.

## Part 3:
We will now start to examine the minimal JOS kernel in a bit more detail.

### Exercise 7. 
>Use QEMU and GDB to trace into the JOS kernel and stop at the movl %eax, %cr0. Examine memory at 0x00100000 and at 0xf0100000. Now, single step over that instruction using the stepi GDB command. Again, examine memory at 0x00100000 and at 0xf0100000. Make sure you understand what just happened.

>What is the first instruction after the new mapping is established that would fail to work properly if the mapping weren't in place? Comment out the movl %eax, %cr0 in kern/entry.S, trace into it, and see if you were right.

movl %eax, %cr0 enables paging. After this instruction, memory at 0x00100000 and at 0xf0100000 becomes the same. The first failed instruction if the mapping weren't in place is jmp *%eax. This is because %Eax stores 0xf010002c, an address outside of RAM.

### Exercise 8. 
> We have omitted a small fragment of code - the code necessary to print octal numbers using patterns of the form "%o". Find and fill in this code fragment.

```{r}
vprintfmt:
    case 'o':
        num = getuint(&ap, lflag); // get the number;
    base = 8;
    goto: number; // print the number with the corresponding base;
```
> Explain the interface between printf.c and console.c. Specifically, what function does console.c export? How is this function used by printf.c?

In /inc/stdio.h, console.c export cputchar, getchar, iscons. printf.c uses void cputchar(int c) in console.c. In addition to cputchar, it contains a position pointer.

> Explain the following from console.c:
```{r}
1      if (crt_pos >= CRT_SIZE) {
2              int i;
3              memmove(crt_buf, crt_buf + CRT_COLS, (CRT_SIZE - CRT_COLS) * sizeof(uint16_t));
4              for (i = CRT_SIZE - CRT_COLS; i < CRT_SIZE; i++)
5                      crt_buf[i] = 0x0700 | ' ';
6              crt_pos -= CRT_COLS;
7      }
```
If the current position is bigger than the screen size, move the whole screen content up a line and fill the last line with '   '. Repoint the pointer to the first position of the last line.

> For the following questions you might wish to consult the notes for Lecture 2. These notes cover GCC's calling convention on the x86. Trace the execution of the following code step-by-step:

```{r}
int x = 1, y = 3, z = 4;
cprintf("x %d, y %x, z %d\n", x, y, z);
```

> In the call to cprintf(), to what does fmt point? To what does ap point?
List (in order of execution) each call to cons_putc, va_arg, and vcprintf. For cons_putc, list its argument as well. For va_arg, list what ap points to before and after the call. For vcprintf list the values of its two arguments.

In the call to cprintf(), fmt = "x %d, y %x, z %d\n", ap = "x %d, y %x, z %d\n".
First vcprintf is called with fmt = "x %d, y %x, z %d\n", ap = "\001". Then for every character in fmt except '%', cons_putc is called with the ASCII code of that character, va_arg is called after escaping with '%'. ap changed from "\001" to "\003" and "\004" after va_arg being called.

> Run the following code.
```{r}
    unsigned int i = 0x00646c72;
    cprintf("H%x Wo%s", 57616, &i);
```
    
> What is the output? Explain how this output is arrived at in the step-by-step manner of the previous exercise. Here's an ASCII table that maps bytes to characters.
The output depends on that fact that the x86 is little-endian. If the x86 were instead big-endian what would you set i to in order to yield the same output? Would you need to change 57616 to a different value?

The output is "He110 World". In ASCII table, 0x72 = 'r', 0x6c = 'l', 0x64 = 'd'. If it is big-endian, then we should set i to be 0x726c6400.

TODO.

> In the following code, what is going to be printed after 'y='? (note: the answer is not a specific value.) Why does this happen? 

```{r}
    cprintf("x=%d y=%d", 3);
 ```
   y = -267321364. cprintf calls cprintfmt(..., ap). After printing "3", ap points to the next address that stores the 0xf010ffec, -267321364 in decimal.
 
> Let's say that GCC changed its calling convention so that it pushed arguments on the stack in declaration order, so that the last argument is pushed last. How would you have to change cprintf or its interface so that it would still be possible to pass it a variable number of arguments?

TODO.

### Exercise 9. 
> Determine where the kernel initializes its stack, and exactly where in memory its stack is located. How does the kernel reserve space for its stack? And at which "end" of this reserved area is the stack pointer initialized to point to?

In entry.S, the kernel initializes its stack at bootstacktop. The stack pointer initialized to point to is the highest end.

### Exercise 10. 
> To become familiar with the C calling conventions on the x86, find the address of the test_backtrace function in obj/kern/kernel.asm, set a breakpoint there, and examine what happens each time it gets called after the kernel starts. How many 32-bit words does each recursive nesting level of test_backtrace push on the stack, and what are those words?

In each call of test_backtrace, %ebp and %ebx are pushed on the stack. %ebp is the previous function's base pointer and %ebx is the previous function's variable. 

### Exercise 11
>  The backtrace function should display a listing of function call frames in the following format:
```{r}
Stack backtrace:
ebp f0109e58  eip f0100a62  args 00000001 f0109e80 f0109e98 f0100ed2 00000031
ebp f0109ed8  eip f01000d6  args 00000000 00000000 f0100058 f0109f28 00000061
```
> Implement the backtrace function as specified above. Use the same format as in the example, since otherwise the grading script will be confused. When you think you have it working right, run make grade to see if its output conforms to what our grading script expects, and fix it if it doesn't. After you have handed in your Lab 1 code, you are welcome to change the output format of the backtrace function any way you like.

kern/monitor.c
```{r}
int
mon_backtrace(int argc, char **argv, struct Trapframe *tf)
{
    // Your code here.
    
    uint32_t* ebp = (uint32_t*)read_ebp();
    cprintf("Stack backtrace:\n");
    while(ebp != 0){
        cprintf("  ebp %08x  ", ebp); //print ebp
        cprintf("eip %08x  args", *(ebp+1)); //print the return address
        for(int i = 2; i <= 6; i++){
            cprintf(" %08x", *(ebp+i)); //print the five arguments pushed before
        }
        ebp = (uint32_t*)*ebp;
        cprintf("\n");
    }
    return 0;
}
```

### Exercise 12. 
>Modify your stack backtrace function to display, for each eip, the function name, source file name, and line number corresponding to that eip.

kern/kdebug.c
```{r}
stab_binsearch(stabs, &lline, &rline, N_SLINE, addr);
    if(rline < lline) return -1;
    info->eip_line = stabs[lline].n_desc;
```

kern/monitor.c
```{r}
int
mon_backtrace(int argc, char **argv, struct Trapframe *tf)
{
    // Your code here.
    
    uint32_t* ebp = (uint32_t*)read_ebp();
    cprintf("Stack backtrace:\n");
    struct Eipdebuginfo info;
    while(ebp != 0){
        cprintf("  ebp %08x  ", ebp); //print ebp
        cprintf("eip %08x  args", *(ebp+1)); //print the return address
        for(int i = 2; i <= 6; i++){
            cprintf(" %08x", *(ebp+i)); //print the five arguments pushed before
        }
        debuginfo_eip((uintptr_t)ebp[1], &info);
        cprintf("\t%s:", info.eip_file);
        cprintf("%d: ", info.eip_line);
        cprintf("%.*s+", info.eip_fn_namelen, info.eip_fn_name);
        cprintf("%d\n", ebp[1]-info.eip_fn_addr);

        ebp = (uint32_t*)*ebp;
        cprintf("\n");
    }
    return 0;
}
```
