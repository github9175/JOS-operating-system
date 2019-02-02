# JOS-operating-system

## Lab 3 notes: User Environments

The kernel uses the Env data structure to keep track of each user environment.

Once JOS gets up and running, the envs pointer points to an array of Env structures representing all the environments in the system.

The JOS kernel keeps all of the inactive Env structures on the env_free_list.

The kernel uses the curenv symbol to keep track of the currently executing environment at any given time. 

Like a Unix process, a JOS environment couples the concepts of "thread" and "address space".

Because we do not yet have a filesystem, we will set up the kernel to load a static binary image that is embedded within the kernel itself. JOS embeds this binary in the kernel as a ELF executable image.

env_init()
Initialize all of the Env structures in the envs array and add them to the env_free_list. Also calls env_init_percpu, which configures the segmentation hardware with separate segments for privilege level 0 (kernel) and privilege level 3 (user).

env_setup_vm()
Allocate a page directory for a new environment and initialize the kernel portion of the new environment's address space.

region_alloc()
Allocates and maps physical memory for an environment

load_icode()
You will need to parse an ELF binary image, much like the boot loader already does, and load its contents into the user address space of a new environment.
