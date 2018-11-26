# JOS-operating-system

## Lab 1 notes

Exercise 8

kern/printf.c, kern/console.c, lib/printfmt.c

- printf.c
  - vcprintf uses vprintfmt in printfmt.c
  - putch uses cputchar in console.c (output data to console)

- printfmt.c (format printing)
  - vprintfmt uses putch in printf.c

Exercise 11 & 12

- kern/kdebug.c
  - debuginfo_eip(addr, struct Eipdebuginfo *info) uses stab_binsearch(stabs, &lline, &rline, N_SLINE, addr) to find debug information.

- kern/monitor.c
  - mon_backtrace() uses read_ebp() and debuginfo_eip(eip, &info) to print trace information.
  
