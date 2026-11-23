basically they use UD2 for a anti-reverse engineering technique. The UD2 instruction is an undefined instruction in x86 architecture that causes the CPU to generate an invalid opcode exception when executed. 

but then in the main func, the logic is designed to catch this exception and let the func 'segill_sigaction' handle it.

check the 'segill_sigaction' func, it basically move the register that holds value of the instruction pointer (RIP) to the next instruction after the UD2, effectively skipping over the UD2 instruction and allowing the program to continue executing normally.

