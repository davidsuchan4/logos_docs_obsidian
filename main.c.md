includes [[console.h]], [[fs.h]], [[fs_types.h]], [[loader.h]], [[trap.h]], [[syscall.h]], [[process.h]],  and [[seed_filesystem.h]]

## Declarations

TRAP_STACK_SIZE
trap_stack (array of bytes)
program_should_exit (flag to signify whether the top_level process has exited - default 0)

## c_trap_handler

Called by the [[assembly trap handler]]

It dispatches system calls and handles other traps

We first get the reason for the trap through [[get_mcause()]] (currently only has support for [[ecalls]])

#### Case: Ecall

[[syscall_dispatch()]] is called on the trap frame  - if the process was a top level process i.e process slot 0 (the shell) called [[sys_exit]] then we change the program_should_exit flag to 1. We then call [[restore kernel context]] which gives control back to the kernel which spawns another shell process in process slot 0.

In any other case the process goes through these steps:
 - a snapshot of the [[registers]] is taken as they were when the [[ecall]] happened and saved into a [[trap frame]].
 - control of the [[CPU]] is then given over to the [[kernel]]
 - the kernel executes the requested [[system call]]
 - once the system call is completed the kernel surrenders control of the cpu and hands it back to the process
 - the process restores its saved registers while only changing the mepc to move after the ecall instruction and the a0 register to hold the result of the ecall by calling [[trap_ret()]]

#### Case: Interrupt

Currently the trap handler cannot handle interrupts and prints an error message and attempts to return to the process through [[trap_ret()]]

#### Case: Exception

Currently the trap handler cannot handle exceptions and prints an error message before stalling



### setup_trap_frame

Creates a synthetic trap frame for a process from which it then returns in order to take control of the cpu

This is done by initializing the trap frames [[stack pointer]] and setting the address of the [[c_trap_handler]] of the trap frame to [[c_trap_handler()]]


### main

Main is the entry point into the kernel from the startup flow. It initialises everything the kernel needs to run i.e the [[console device]], [[filesystem]], [[process table]], and [[shell]]

[[console_dev_init()]] is called in order to initialise the console device so that we have I/O

[[proc_init()]] is called which initialises the process table 

[[block_init()]] initialises the logosfs partition in flash

it attempts to mount the filesystem through [[fs_mount()]] 
 - if an error is thrown then the filesystem is formatted through [[fs_format()]]
 - after which a mount is once again attempted - on failure an error message is printed and the program stalls


[[logos_seed_filesystem()]] is called which creates the filesystem structure/tree

We start a loop in which we continuosly try to run the [[shell]] which ensures that it restarts after every shell exit

We first reserve a new [[process slot]] for the shell (it is always process 0)
A new [[process struct]] is created to hold all the information about the shell
We then assign the slot we reserved to this struct
The environment variables are initialised by [[proc_env_init()]] and [[proc_set_env_int()]]
The shell is loaded into this slot by the [[elf_loader]] using [[elf_load_at()]]
A synthetic trap frame is set up that the program will then return from in order to take control of the CPU by way of [[setup_trap_frame()]] 
then we set the relevant registers within the trap frames to our desired values
- we set the [[mepc]] (program counter) to the entry point of our elf
- this is the first process so we are not expecting to return anywhere hence we set the [[ra]] to 0
- we set the [[trap frame stack pointer]] to the top of the [[processes stack]]
- we are not passing any arguments so both [[a0]] and [[a1]] are left as null
- We set the [[state of the process]] as running
- the fd of the parent is set to -1 since this is the first process it has no parent 

We then initialise the [[file descriptors]] used for the console through [[proc_fd_init]]

Finally we start the shell by allocating the shells slot as the current process and setting [[program_should_exit]] to 0
we set the address of the trap handler in [[mtvec]] through [[set_trap_handler()]]
[[run_user_program()]] runs the shell 

on shell exit we save the exit code and print it before freeing the process slot and restarting the shell


