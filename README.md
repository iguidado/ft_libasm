# ft_libasm
libasm.  all things wrote here is only applicable to x86_64 NASM on linux wich is what you need if you want to pass your project

first thing is compilation {"but we didn't write code"} sure young padawan but it will go fast so don't worry

With higher level langage (HLL) translation pass by 3 phase depending on tool,  
-- compilation translate HLL in Assembly into ASM file {*.s|*.asm}, 
-- Assembler translate ASM in Machine langage into object file {*.o}
-- Linkage link all object file into one executable file

As there is multiple type of executable file depending of the system we got to specify type to the Assembler

The classic assembler we use for NASM is..... nasm  and we  use 2 option : 
-- "-f elf64" to do 64 bit Linux executable file
-- "-f macho64" to do 64 bit Mac executable file

e.g. -->  nasm -felf64 my_first.asm -o my_first.o


but since we use a C writted code for call we need to use Clang or GCC for all the operation

We will also use compiler to translate C code into assembly since all call must be done from a C file

------------------------

```
/*	
**		
**	this part doesn't work and i don't know why
**
**	   you can use both gcc and clang (3.5+) with "-S masm=intel" to generate NASM file
**	
**	   e.g. --> clang -S masm=intel main.c -o main.asm
**	
**	   you can also use the dedicated tool for compilation from clang which is llc (but flags are a bit more rough to memorize)
**	
**	   e.g  -->  llc main.c --x86-asm-syntax=intel -o main.s
**	
*/
```
------------------------

for the link part we are going to use clang (or gcc) since classic ASM linker "ld" won't understand some of C assembled instruction
you should already know how to do it but here is some reminder

clang main.o my_first_asm.o [...] -o a.out
clang main.o libasm.a             -o a.out


you can use the Makefile as a reference if you are lost

=============================================================
many things to learn in this project (as always)

memory management
		-register
		-stack
		-memory

Register:   They are the memory of the processor. which are the fastest memory
			available to read and write for (you guess it?) the processor.
			these register will be used for syscall and 
			general procedure management (loop, conditional jump, etc.)

all register are of ABC form with : A being optional is the length of the register (A='r' for 64bits, A='e' for 32bits and A='' for classic 16bit use
									 B is simply a letter to differenciate a register in the same classe (like B='a', B='b', b='c', B='d' for the C='x' being the general usage register)
									 C is for the type of the register as said above C='x' is for general arithmetic operation, C='i' usualy used to old string address (think index)
										C='p' is for stack's pointer

general use register : you can freely use these register to stock variable but these register also have a specifique use in the langage so remember it when making operation

	rax = it is the register used to determine which system function use with instruction "syscall" e.g. '60' is for exit, '0' is for read... more in the syscall: part
		  it is also used by syscall to return the resulting value (So syscall replace it's calling value to it's return value alright)
		  it is also used by the "RET" instruction as it will be the value returned to caller (usually a HLL function/program)
	rbx = it is used 
	rcx = it is used by the "loop" instruction to determine when to stop the loop, it is decremented at each loop

	rdi = when using a syscall it is the first argument, it is also the first argument when you make an NASM call from C
	rsi = when using a syscall it is the second argument, it is also the second argument when you make an NASM call from C

	example -->  write(1, "hello world\n", 12); --> rdi = 1 ; rsi = "hello world", 10, 0 ; rdx = 12

Notice how we changed the \n to 13 and the NULL terminating byte to 0 as ASM don't translate special symbol so i gave you their numerical value as it is how ASM see them

instruction: There is a shitload of instruction in ASM, but there are general rules that all instruction follow
			 for operation in NASM attribution of value goes from RIGHT to LEFT, contrary to GAS.
			 so an operation like MUL rax, rbx =   rax <-- rax * rbx
			 same for all operation taking 2 operands
			the instruction can be broken in 3 family : arithmetiaque operation (attribution "MOV", multiply "MUL", addition "ADD", substraction "SUB) which follow rule above

====================================
RAM memory

RAM is where is located all the memory that don't fit in the register
it is composed of :

	[Stacks]
	   |
	   V
	   ^
	   |
	[ Heap ]
	--------
	[ Bss  ]
	[ Data ]
	[ Code ]


Stacks

So what is Stack ? stack is a memory structure used to stock data when binary is running.
Stack is managed like a pile : you put variable on top of each other and get it back is done from up to bottom
Stack adress start with it's highest value meaning each data added got a lower adress in memory
It is a FILO struct meaning data
it perticularity is that his size is variable meaning you can stock more or


=====================================

Call

---------------------------------------------------

Calling Convention :
	Each flavor of ASM  got his own calling convention used to make a call 
	(syscall, extern call...), it is also different between 32|64 bit version

	in intel NASM 64 bit it is used as next :  
		call(rdi, rsi, rbx, rcx

---------------------------------------------------

syscall:

	exit(exit_code)							,	60
	read(fd_to_read, buffer, nb_of_bit)		,	0
	write(fd, string_to_write, nb_of_bit)	,	1

