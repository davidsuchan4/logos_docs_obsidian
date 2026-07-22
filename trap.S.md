
The section that the trap handler will be placed in is a special part of memory reserved by the esp specifically for the trap handler called [[.exception_vectors.text]] it is set to be exectuable ("ax")

Current settings are saved by [[.option push]] and the compiler is told not to optimise or shorten instructions everything should be executed explicitly as it is written

`.global _logos_trap_stack` marks the beginning of the trap handler for the linker so that other files can resolve the symbol
`_logos_trap_start:` defines an assembly label that assembly code can jump to

## set_trap_handler

creates the symbol for the linker, identifies it as a function and declares the assembly jump label
``
```
.global set_trap_handler
.type set_trap_handler, @function
set_trap_handler:
```

`ori a0, a0, 1` performs a bitwise OR operation between a0 and 1 which forces the lowest bit of a0 to be 1 this sets the trap mode to be vectored which allows for different trap handlers for different interrupts

`csrw mtvec, a0` it takes the recently changed address from a0 and writes it to mtvec using a special csr command for writing to special registers - this stores the address of the trap handler in [[mtvec.]] The original address was stored in a0 since that is the register used for passing arguments

`csrw mscratch, a1` a1 contains a pointer to the address of the current processes trap frame - its loaded into mscratch so that the trap handler knows where to save the processes registers in the case of an interrupt

## set_mie

`csrw mie, a0` sets the bit mask in the [[mie]] register 

## get_mtvec

`csrr a0, mtvec` loads the address of the [[trap vector table]] into the return register

## get_mie

`csrr a0, mie` loads the currently allowed interrupts into the return register

## set_mstatus_bit

`csrw mstatus, a0` sets every [[mstatus register]] bit selected by the mask passed by the caller in a0

## enable_interrupts

`csrsi mstatus, 8` since bit 3 in mstatus controls global interrupts we set it to 1 which enables global interrupts

## disable_interrupts

`crci matatus, 8` since bit 3 in mstatus controls global interrupts we clear it (set it to 0) which disables global interrupts

## get_mcause

`csrr mcause, a0` loads the reason for the interrupt from the [[mcause register]] into the return register

## kernel_vector_table

this function is the one that sits in mtvec and is what is called upon an interrupt. it will call a trap handler based on the type of interrupt it recieves

`.balign 0x100` this means that the address of this function has to be divisible by 100 - this is required by the esp idf build
`j trap handler` currently there is only one trap handler so it jumps to it no matter what


## trap_handler

This is currently the only trap handler and is the function that is called when an [[interrupt]] occurs. It saves all of the processes registers into the processes trap frame and loads the kernels context back into the required registers so that it may perform the system call

`csrrw a0, mscratch, a0`  this loads the pointer to the trap frame into a0 and moves the value previously held in a0 temporarily into mscratch

```
sw ra,  TF_RA_OFFSET(a0)
sw sp,  TF_SP_OFFSET(a0)
sw gp,  TF_GP_OFFSET(a0)
sw tp,  TF_TP_OFFSET(a0)
sw t0,  TF_T0_OFFSET(a0)
sw t1,  TF_T1_OFFSET(a0)
sw t2,  TF_T2_OFFSET(a0)
sw s0,  TF_S0_OFFSET(a0)
sw s1,  TF_S1_OFFSET(a0)
sw a1,  TF_A1_OFFSET(a0)
sw a2,  TF_A2_OFFSET(a0)
sw a3,  TF_A3_OFFSET(a0)
sw a4,  TF_A4_OFFSET(a0)
sw a5,  TF_A5_OFFSET(a0)
sw a6,  TF_A6_OFFSET(a0)
sw a7,  TF_A7_OFFSET(a0)
sw s2,  TF_S2_OFFSET(a0)
sw s3,  TF_S3_OFFSET(a0)
sw s4,  TF_S4_OFFSET(a0)
sw s5,  TF_S5_OFFSET(a0)
sw s6,  TF_S6_OFFSET(a0)
sw s7,  TF_S7_OFFSET(a0)
sw s8,  TF_S8_OFFSET(a0)
sw s9,  TF_S9_OFFSET(a0)
sw s10, TF_S10_OFFSET(a0)
sw s11, TF_S11_OFFSET(a0)
sw t3,  TF_T3_OFFSET(a0)
sw t4,  TF_T4_OFFSET(a0)
sw t5,  TF_T5_OFFSET(a0)
sw t6,  TF_T6_OFFSET(a0)

```

All of these commands store the values previously held in the registers into the correct offset of the address held in a0 which is the address of the trap frame

```
csrr t0, mepc
sw t0, TF_MEPC_OFFSET(a0)
csrr t0, mstatus
sw t0, TF_MSTATUS_OFFSET(a0)
csrr t0, mcause
sw t0, TF_MCAUSE_OFFSET(a0)
csrr t0, mtval
sw t0, TF_MTVAL_OFFSET(a0)

csrr t0, mscratch
sw t0, TF_A0_OFFSET(a0)
```

The special csr registers cant be easily directly written to the trap frame so we first load it into t0 which has already been saved into the trap frame and load it into the offset corresponding to the special register

`lw sp, TF_C_TRAP_SP_OFFSET(a0)` we set the stack pointer to point to the top of the [[kernel trap stack]]

```
la t0, kernel_context
lw gp, 4(t0)
lw tp, 8(t0)
```

here we load the registers that the kernel saved before the user program took control of the CPU -  it can be thought of like a very small stack frame for the kernel

```
lw t0, TF_C_TRAP_OFFSET(a0)
jalr ra, t0, 0
```

here we load the address of the c trap handler into t0
we then set the return address to the current return address + 4 so that we dont end up in a loop after which we move the [[pc]] to the address stored in t0 + 0 and the [[c_trap_handler]] is executed

```
la t0, kernel_context
lw t0, 0(t0)
j restore_kernel_context
```

Now this code is only ever executed if its the shell process that has been exited in [[c_trap_handler]] and in that case we need to give back control to the kernel so that it may start another shell process

first we load the address of kernel_context into t0
then the first value at that address is loaded into t0 overwriting its previous value 
we then load all of the kernels registers that it will need back into the registers of the cpu through [[restore_kernel_context]] so that it can restart the shell


## trap_ret

```
lw t0, TF_MEPC_OFFSET(a0)
csrw mepc, t0
```

the stored mepc is loaded into t0
the saved mepc from the trap frame is loaded into the mepc register so that the cpu knows where to resume execution

```
lw t0, TF_MSTATUS_OFFSET(a0)
ori t0, t0, MSTATUS_MPIE
csrw mstatus, t0
```

the stored mstatus is loaded into t0
a bitwise or is done between between the stored [[MPIE]] and the stored mstatus which as this point is in t0
we then load this correctly configured mstatus into the mstatus register

```
lw t0, TF_A0_OFFSET(a0)
csrw mscratch, t0
```

we load the stored a0 into t0 which is then subsequently loaded into the mscratch register (this is the saved a0 from the original process the current a0 hold the process's mscratch value i.e the pointer to the trap frame)

```
lw ra,  TF_RA_OFFSET(a0)
lw sp,  TF_SP_OFFSET(a0)
lw gp,  TF_GP_OFFSET(a0)
lw tp,  TF_TP_OFFSET(a0)
lw t0,  TF_T0_OFFSET(a0)
lw t1,  TF_T1_OFFSET(a0)
lw t2,  TF_T2_OFFSET(a0)
lw s0,  TF_S0_OFFSET(a0)
lw s1,  TF_S1_OFFSET(a0)
lw a1,  TF_A1_OFFSET(a0)
lw a2,  TF_A2_OFFSET(a0)
lw a3,  TF_A3_OFFSET(a0)
lw a4,  TF_A4_OFFSET(a0)
lw a5,  TF_A5_OFFSET(a0)
lw a6,  TF_A6_OFFSET(a0)
lw a7,  TF_A7_OFFSET(a0)
lw s2,  TF_S2_OFFSET(a0)
lw s3,  TF_S3_OFFSET(a0)
lw s4,  TF_S4_OFFSET(a0)
lw s5,  TF_S5_OFFSET(a0)
lw s6,  TF_S6_OFFSET(a0)
lw s7,  TF_S7_OFFSET(a0)
lw s8,  TF_S8_OFFSET(a0)
lw s9,  TF_S9_OFFSET(a0)
lw s10, TF_S10_OFFSET(a0)
lw s11, TF_S11_OFFSET(a0)
lw t3,  TF_T3_OFFSET(a0)
lw t4,  TF_T4_OFFSET(a0)
lw t5,  TF_T5_OFFSET(a0)
lw t6,  TF_T6_OFFSET(a0)
```

We restore all the saved registers from the trap frame (as seen we are using offsets from the address stored in a0 )

`csrrw a0, mscratch, a0` this swaps the values between mscratch which holds the saved a0 value and a0 which holds the pointer to the trap frame

`fence.i` makes sure that the cpu acknowledges all the changes made in memory

```
csrr t0, mepc
jr t0
```

the mepc value which is the next instruction after the ecall to be performed in the original process is loaded into t0 - it is then executed and the original process resumes

## run_user_program

This program hands control over control from the kernel to the user process

`    addi sp, sp, -64` first space is allocated on the stack to save the registers

```
sw   ra,  60(sp)
sw   s0,  56(sp)
sw   s1,  52(sp)
sw   s2,  48(sp)
sw   s3,  44(sp)
sw   s4,  40(sp)
sw   s5,  36(sp)
sw   s6,  32(sp)
sw   s7,  28(sp)
sw   s8,  24(sp)
sw   s9,  20(sp)
sw   s10, 16(sp)
sw   s11, 12(sp)
```

then the registers are actually saved

`la t0, kernel_context` the pointer to kernel_context is loaded into t0

```
sw   sp, 0(t0)
sw   gp, 4(t0)
sw   tp, 8(t0)
```

the kernels [[stack pointer]], [[global pointer]] and [[thread pointer]] are all saved into kernel context

`j trap_ret` trap ret is called which hands over control to the user process

## restore_kernel_context

```
mv   sp, t0
lw   s11, 12(sp)
lw   s10, 16(sp)
lw   s9,  20(sp)
lw   s8,  24(sp)
lw   s7,  28(sp)
lw   s6,  32(sp)
lw   s5,  36(sp)
lw   s4,  40(sp)
lw   s3,  44(sp)
lw   s2,  48(sp)
lw   s1,  52(sp)
lw   s0,  56(sp)
lw   ra,  60(sp)
addi sp, sp, 64
li   a0, 0
ret
```

loads all the stored kernel data into the relevant registers

