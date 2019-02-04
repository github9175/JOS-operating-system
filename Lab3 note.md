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
Set envs

env_setup_vm()
Allocate a page directory for a new environment and initialize the kernel portion of the new environment's address space.
e->env_pgdir

region_alloc()
Allocates and maps physical memory for an environment
r = page_insert(e->env_pgdir, p, i, PTE_W | PTE_U);

load_icode()
You will need to parse an ELF binary image, much like the boot loader already does, and load its contents into the user address space of a new environment.

env_create()
Allocate an environment with env_alloc and call load_icode to load an ELF binary into it.

env_run()
Start a given environment running in user mode.
switch from curenv to env e.

ENV from inc/env.h
```
struct Env {
	struct Trapframe env_tf;	// Saved registers
	struct Env *env_link;		// Next free Env
	envid_t env_id;			// Unique environment identifier
	envid_t env_parent_id;		// env_id of this env's parent
	enum EnvType env_type;		// Indicates special system environments
	unsigned env_status;		// Status of the environment
	uint32_t env_runs;		// Number of times environment has run

	// Address space
	pde_t *env_pgdir;		// Kernel virtual address of page dir
};
```

ELF from inc/elf.h
```
struct Elf {
	uint32_t e_magic;	// must equal ELF_MAGIC
	uint8_t e_elf[12];
	uint16_t e_type;
	uint16_t e_machine;
	uint32_t e_version;
	uint32_t e_entry;
	uint32_t e_phoff;
	uint32_t e_shoff;
	uint32_t e_flags;
	uint16_t e_ehsize;
	uint16_t e_phentsize;
	uint16_t e_phnum;
	uint16_t e_shentsize;
	uint16_t e_shnum;
	uint16_t e_shstrndx;
};
```

Proghdr from inc/elf.h
```
struct Proghdr {
	uint32_t p_type;
	uint32_t p_offset;
	uint32_t p_va;
	uint32_t p_pa;
	uint32_t p_filesz;
	uint32_t p_memsz;
	uint32_t p_flags;
	uint32_t p_align;
};
```
