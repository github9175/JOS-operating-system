# JOS-operating-system

## Lab 3 notes: User Environments

The kernel uses the Env data structure to keep track of each user environment.

Once JOS gets up and running, the envs pointer points to an array of Env structures representing all the environments in the system.

The JOS kernel keeps all of the inactive Env structures on the env_free_list.

The kernel uses the curenv symbol to keep track of the currently executing environment at any given time. 

Like a Unix process, a JOS environment couples the concepts of "thread" and "address space".
