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

elf to e

env_create()
Allocate an environment with env_alloc and call load_icode to load an ELF binary into it.

load_icode(e, binary);



env_run()
Start a given environment running in user mode.
switch from curenv to env e.

set curenv

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

You will now need to implement basic exception and system call handling, so that it is possible for the kernel to recover control of the processor from user-mode code.

Exceptions and interrupts are both "protected control transfers," which cause the processor to switch from user to kernel mode (CPL=0) without giving the user-mode code any opportunity to interfere with the functioning of the kernel or other environments.

In order to ensure that these protected control transfers are actually protected, the processor's interrupt/exception mechanism is designed so that the code currently running when the interrupt or exception occurs does not get to choose arbitrarily where the kernel is entered or how. Instead, the processor ensures that the kernel can be entered only under carefully controlled conditions. On the x86, two mechanisms work together to provide this protection:

trapentry.S
GD_KD

Trapframe from inc/trap.h
```
struct Trapframe {
	struct PushRegs tf_regs;
	uint16_t tf_es;
	uint16_t tf_padding1;
	uint16_t tf_ds;
	uint16_t tf_padding2;
	uint32_t tf_trapno;
	/* below here defined by x86 hardware */
	uint32_t tf_err;
	uintptr_t tf_eip;
	uint16_t tf_cs;
	uint16_t tf_padding3;
	uint32_t tf_eflags;
	/* below here only when crossing rings, such as from user to kernel */
	uintptr_t tf_esp;
	uint16_t tf_ss;
	uint16_t tf_padding4;
} __attribute__((packed));
```
Setgate from inc/mmu.h
```
// Set up a normal interrupt/trap gate descriptor.
// - istrap: 1 for a trap (= exception) gate, 0 for an interrupt gate.
    //   see section 9.6.1.3 of the i386 reference: "The difference between
    //   an interrupt gate and a trap gate is in the effect on IF (the
    //   interrupt-enable flag). An interrupt that vectors through an
    //   interrupt gate resets IF, thereby preventing other interrupts from
    //   interfering with the current interrupt handler. A subsequent IRET
    //   instruction restores IF to the value in the EFLAGS image on the
    //   stack. An interrupt through a trap gate does not change IF."
// - sel: Code segment selector for interrupt/trap handler
// - off: Offset in code segment for interrupt/trap handler
// - dpl: Descriptor Privilege Level -
//	  the privilege level required for software to invoke
//	  this interrupt/trap gate explicitly using an int instruction.
#define SETGATE(gate, istrap, sel, off, dpl)			\
{								\
	(gate).gd_off_15_0 = (uint32_t) (off) & 0xffff;		\
	(gate).gd_sel = (sel);					\
	(gate).gd_args = 0;					\
	(gate).gd_rsv1 = 0;					\
	(gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;	\
	(gate).gd_s = 0;					\
	(gate).gd_dpl = (dpl);					\
	(gate).gd_p = 1;					\
	(gate).gd_off_31_16 = (uint32_t) (off) >> 16;		\
}
```
trap_init->traphandler->all_trap->trap->trap_dispatch->page_fault_handler,monitor,syscall->kern/syscall:

lib/entry.S->lib/libmain.c libmain()->umain()

user_mem_check？

In all three cases, the operating system design must arrange for the following to
happen. The system must save the processor’s registers for future transparent resume.
The system must be set up for execution in the kernel. The system must chose a place
for the kernel to start executing. The kernel must be able to retrieve information about
the event, e.g., system call arguments. It must all be done securely; the system must
maintain isolation of user processes and the kernel.

The basic plan is as follows. An interrupts stops the normal processor loop and
starts executing a new sequence called an interrupt handler. Before starting the interrupt handler, the processor saves its registers, so that the operating system can restore
them when it returns from the interrupt. A challenge in the transition to and from
the interrupt handler is that the processor should switch from user mode to kernel
mode, and back.

The x86 has 4 protection levels, numbered 0 (most privilege) to 3 (least privilege).
In practice, most operating systems use only 2 levels: 0 and 3, which are then called
kernel mode and user mode, respectively. The current privilege level with which the
x86 executes instructions is stored in %cs register, in the field CPL.
On the x86, interrupt handlers are defined in the interrupt descriptor table (IDT).
The IDT has 256 entries, each giving the %cs and %eip to be used when handling the
corresponding interrupt.
To make a system call on the x86, a program invokes the int n instruction, where
n specifies the index into the IDT. The int instruction performs the following steps:
• Fetch the n’th descriptor from the IDT, where n is the argument of int.
• Check that CPL in %cs is <= DPL, where DPL is the privilege level in the descriptor.
• Save %esp and %ss in CPU-internal registers, but only if the target segment selector’s PL < CPL.
• Load %ss and %esp from a task segment descriptor.
• Push %ss.
• Push %esp.
• Push %eflags.
• Push %cs.
• Push %eip.
• Clear the IF bit in %eflags, but only on an interrupt.
• Set %cs and %eip to the values in the descriptor

Figure 3-1 shows the stack after an int instruction completes and there was a
privilege-level change (the privilege level in the descriptor is lower than CPL). If the
int instruction didn’t require a privilege-level change, the x86 won’t save %ss and
%esp. After both cases, %eip is pointing to the address specified in the descriptor table, and the instruction at that address is the next instruction to be executed and the
first instruction of the handler for int n. It is job of the operating system to implement these handlers, and below we will see what xv6 does.
An operating system can use the iret instruction to return from an int instruction. It pops the saved values during the int instruction from the stack, and resumes
execution at the saved %eip.
