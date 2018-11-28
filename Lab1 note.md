# JOS-operating-system

## Lab 1 notes: Booting a PC

Exercise 8

- kern/printf.c
  - vcprintf uses vprintfmt in lib/printfmt.c
  - putch uses cputchar in kern/console.c (output data to console)

- lib/printfmt.c (format printing)
  - vprintfmt uses putch in kern/printf.c

Exercise 11 & 12

- kern/kdebug.c
  - debuginfo_eip(addr, struct Eipdebuginfo *info) uses stab_binsearch(stabs, &lline, &rline, N_SLINE, addr) to find debug information.

- kern/monitor.c
  - mon_backtrace() uses read_ebp() and debuginfo_eip(eip, &info) to print trace information.
  
