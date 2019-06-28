### Implement simple commands, such as:

$ ls

The parser already builds an execcmd for you, so the only code you have to write is for the ' ' case in runcmd.

### I/O redirection
Implement I/O redirection commands so that you can run:

echo "6.828 is cool" > x.txt
cat < x.txt
The parser already recognizes ">" and "<", and builds a redircmd for you, so your job is just filling out the missing code in runcmd for those symbols. You might find the man pages for open and close useful.

Note that the mode field in redircmd contains access modes (e.g., O_RDONLY), which you should pass in the flags argument to open; see parseredirs for the mode values that the shell is using and the manual page for open for the flags argument.

Make sure you print an error message if one of the system calls you are using fails.

Make sure your implementation runs correctly with the above test input. A common error is to forget to specify the permission with which the file must be created (i.e., the 3rd argument to open).

### Implement pipes
Implement pipes so that you can run command pipelines such as:

$ ls | sort | uniq | wc
The parser already recognizes "|", and builds a pipecmd for you, so the only code you must write is for the '|' case in runcmd. You might find the man pages for pipe, fork, close, and dup useful.

Test that you can run the above pipeline. The sort program may be in the directory /usr/bin/ and in that case you can type the absolute pathname /usr/bin/sort to run sort. (In your computer's shell you can type which sort to find out which directory in the shell's search path has an executable named "sort".)

Now you should be able to run the following command correctly:

6.828$ a.out < t.sh
Make sure you use the right absolute pathnames for the programs.

