# JOS-operating-system

## Lab 1 notes

Exercise 8

kern/printf.c, kern/console.c, lib/printfmt.c

- printf.c
  - vcprintf uses vprintfmt in printfmt.c
  - putch uses cputchar in console.c (output data to console)

- printfmt.c (format printing)
  - vprintfmt uses putch in printf.c
