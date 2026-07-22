logos is a Unix-Like operating system that is currently on version three on [[logisim]] however it is only in version one on the [[ESP32c3]].


## Startup Flow

### Esp


#### Bootloader and logos entry point

Logos has nothing to do with the ESP ROM bootloader since it is flashed onto the ROM directly on the chip and doesn't allow for any modifications. There is however a second stage bootloader that the esp calls and that is modifiable. Logos does not touch this for 2 reasons. 
 - The [[second stage bootloader]] only initialises hardware which is needed regardless of whether its FreeRTOS running on the board or logos so there was in fact no need to change anything about it.
 - The second reason is that the ROM bootloader can only read ESP header files and the base second stage bootloader already compiled as an ESP header file so once again there was no reason to change anything about the file

Now the actual entry point for logos is start_cpu0 - which is called by start_cpu when is calls SYS_STARTUP_FN which is by default weak linked to the default cpu_start_0 however in logos we override this weak link and link it to our own cpu_start0 which in turn disables the watchdog and calls the entry point of logos [[main()]]. 

#### Main()

The [[main()]] function is the entry point for the logos kernel and where all initialisation happens. 

It initialises systems in this order
- console device ([[console_dev_init()]])
- process table ([[proc_init()]])
- filesystem superblock ([[block_init()]]))
- filesystem (it first attempts to just mount it through [[fs_mount()]] however if that doesnt work it formats the filesystem through [[fs_format()]])
- seeds the filesystem with all user process elf files if not already seeded ([[logos_seed_filesystem()]])

After all this basic initialisation is done it starts the shell by running a loop which allows for shell restarts after every time it exits i.e an interrupt happens

it does this in the following way
- Allocates a [[process slot]] using [[proc_alloc()]]
- If there are no free slots it stalls
- it then initialises the process environment with [[proc_env_init()]] and [[proc_set_ent_int()]]
- The elf sh file located at /bin/sh is loaded using [[elf_load_at()]]
- We then set up a synthetic trap frame with all the data needed for the program to run using [[setup_trap_frame()]]
- We initialise the file descriptor using [[proc_fd_init()]]
- We load the trap handler address into mtvec using [[set_trap_handler()]]
- And finally we use [[run_user_program()]] with the synthetic trap frame with the shell processes data to run the shell

The current available shell commands are [[ls]], [[mkdir]], [[rmdir]], [[cat]], [[hello]], [[mknod]], [[rm]], [[env_demo]],  and [[spawn_demo]]


I have just finished getting the [[shell]] running on the esp. In order to do this i had to reconfigure the [[ELF loader]] since the [[memory of the esp]] is quite different to the [[memory used in the logisim emulator to run logos]]. I also had to [[reconfigure the trap handler specifically for the esp]].
We also changed the [[build process]] for logos so it can be built on both systems. I repurposed [[fstool]] so that it [[seeds the files natively]] rather than having to be built on the users system into [[block.bin]].memory of the esp
