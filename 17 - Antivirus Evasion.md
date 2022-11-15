The *Portable Executable* ( #PE) file format is used on Windows OS for executable and object files.  The PE format represent a Windows data structure that details the information necessary for the Windows loader to manage the wrapped executable code including required #DLL, #API imports and exports tables, etc.

*VirusTotal* generates a hash for each unique submission, which i sthen shared with all participating AV vendors.

## Signature-based detection
A signature is a continuous sequency of bytes within malware that uniquely identifies it.  Signature-based AV dtection is mostly considered a *blacklist* technology.  Fairly easily bypassed, but there is rarely a catch-all AV evasion solution.

## Heuristic and Behavioral-based detection
- *Heuristic-based detection* : relies on various rules and algorithms to determine whetehr or not an action is considered malicious. Look for various patterns and program calls that are considered malicious, instead of byte sequences.
- *Behaviour-based detection* : dynamically analyzes the behavior of a binary file. 

# Bypassing AV detection
Falls into 2 broad categories : #On-disk (modifying malicious files physically stored on disk to evade detection) and #in-memory (reducing the possibliity of detection by avoiding the disk entirely). 

## On-disk evasion - Obfuscating files stored on the physical disk
- Packers - originally designed to reduce the size of an executable while maintaining functional equivalency and changing the binary structure.  Using a packer alone is not enough to evade modern AV scanners.
- Obfuscators - reorganize and mutate code in a way that makes it more difficult to reverse-engineer.
- Crypters - cryptographically alters executable code and adds a decrypting stub that restores the orginial code upon execution (happens in-memory)
- SW protectors - *The Enigma Protector* is one of few commercially available tools that can be successfully used to bypass AV products.

## #in-memory ( #PE injection)
This technique is focused on the manipulation of volatile memory instead of writing any files to the disk.
- Remote proces memory injection - inject the payload into anohter valid PE that is not malicious. Leverages Windows #API s. 
	1. Use the *OpenProcess* function to get a valid *HANDLE* **that we have permission to access**. Once memory allocated in the remote process (using *VirtualAllocEx*), use *WriteProcessMemory* to copy malicious payload to allocated space. Finally, executie in memory in a separate thread using *CreateRemoteThread* API. 
- Reflective #DLL injection - load a DLL stored by the attacker in the process memory. Big challenge : *LoadLibrary* does not support loading a DLL from memory --> requires writing your own version of the API that doesn't rely on disk-based DLL.
- Process Hollowing - launching a non-malicious process in a suspended state, then replacing the image with malicious executable, then resuming the process.
- Inline Hooking - modifying memory and introducing a hookto point the execution flow to our malicious code. After execution, flow returns back to the modified function to appear as if only original code was executed.
- PowerShell in-memory injection - Similar to Remote process memory injection, but targeting the currently executing process (PowerShell interpreter).
	- ***side note*** - by executing a script rather than a PE, it's more difficult for AV manufacturers to determine if the script is maliciious or not as it's run inside an interpreter and the script itself isn't executable cod. 
- #Shellter (shell7er) - Free; good for dynamic shellcode injection by essentially backdooring a valid, non-malicious executable with a malicious shellcode payload. (**apt install shellter**).  Designed to run on Windows.