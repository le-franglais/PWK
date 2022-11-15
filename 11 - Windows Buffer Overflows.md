# Discovering the vulnerability
Primary techniques for this include source code review (easiest if available), reverse engineering, or fuzzing.
## #Fuzzing 
Feeding input fields hoping for unexpected behavior or an application crash.

# Win32 BOF
- Protection mechanisms
	- #DEP (Data Execution Prevention) - HW and SW techs that perform addtl checks on memory to help prevent malicious code from running on a system.  Raises an exception when code execution from data pages is attempted.
	- #ASLR (Address Space Layout Randomization) - randomizes the base addresses of loaded applications and DLLs every time the OS is booted. Very strong when coupled with DEP.
	- #CFG (Control Flow Guard) - performs validation of indirect code branching, preventing overwrites of function pointers.
- Controlling the EIP - determine exactly what part of your buffer is landign in the EIP by using metasploit's **pattern_create.rb**, located in **/usr/share/metasploit-framework/tools/exploit/**. Can also run **msf-pattern_create** from the command line.
- Locating space for the shellcode 
- Checking for #badChars
- Redirecting the execution flow
- Generating #shellcode with #msfvenom