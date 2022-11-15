# x86 Architecture
*Note* : when referring to #Assembly (asm), we're talking aobut an extremely low-level programming language that corresponds very closely to the CPUs built-in machine code instructions.

## Program #memory
Memory addresses in Windows applications range from 0x00000000 (lowest) to 0x7FFFFFFF (highest)
![[Pasted image 20221025164058.png]]

" #theStack" -- a short-term data area for functions, local variables, and program control information.  Each thread in a running application has its own stack.  Last-In-First-Out (LIFO) structure. x86-asm uses *PUSH* and *POP* instructtions to add or remove data from the stack. The return address for different fucntions (along with their parameters and local variables) is stored on the stack

## CPU #registers
9 small, extermely high-speed CPU storage lcoations where data can b efficiently read or maniuplated
![[Pasted image 20221025165007.png]]
General purpose registers -- used to store temporary data : 
- EAX (Accumulator) - arithmetical and logical instructions
- EBX (Base) - base pointer for memory addresses
- ECX (Counter) - loop, shift, and rotation counter
- EDX (data) - I/O port addressing, multiplication, division
- ESI (source index) - pointer addressing of data and source in string copy operations
- EDI (destination index) - pointer addressing of data and destination in string copy operations
#ESP -- The Stack Pointer.  Keeps "track" of the most recently referenced location on the stack (*top of the stack*) by storing a pointer to it.
EBP -- The Base Pointer.  Stores a pointer to the top of the stack when a *function* is called so a function can easily reference information from its own stack frame.
#EIP -- The Instruction Pointer. ***ALWAYS*** points to the next code instruction to be executed. Essentially directs the flow of a program.






