# Information Gathering
The first thing to do once you get a foothold is #enumeration of your environment.  This can be done manually or can be automated.

## Manual enumeration
- Enumerating Users
	- Enumerating current user context - (**whoami** is good for Windows ***AND*** Linux).  Once you have the user name, on Windows you can run **net user** on that, and **id** on Linux systems. 
![[Pasted image 20221028100433.png]]
	- Enumerating other user accounts on the system - Use the **net user** command on Windows and **cat /etc/passwd** on Linux maachines. 
		- ***hint*** : if you see *www-data* as a user, it's likely that a web server is installed
- Enumerating the #hostName : Use the **hostname** command on both Windows ***AND*** Linux.
- Enumerating OS and Arch : 
	- Use the **systeminfo** command on Windows.  To futher refine this, you can use **findstr** command (**systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"**) ![[Pasted image 20221028101337.png]]  
	- On Linux, you can **cat** the **/etc/issue** and **/etc/``*-release``** files, or simply issue the **uname -a** command.
- Enumerating running #process and #services : 
	- on Windows you can run **tasklist** or **tasklist /svc** commands to see processes that are mapped to a specific Windows service.  
	- On Linux, you can run the **ps -aux** command to see processes (w/ or w/o a *tty*) in user-readable format 
- Enumerating network info : 
	- On Windows, run the **ipconfig /all** command to see full TCP/IP configuration of all adapters.  To see routing tables, run the **route print** command.  Finally, run the **netstat -nao** to view active network connection data. 
	- On Linux, run **ifconfig** or **ip a** commands to see TCP/IP config of all adapters. Display routing tables with **route** or **routel** commands.  You can also read **/sbin/route**. Display active network connections  with **netstat** or **ss -anp**. 
- Enumerating #Firewall Status and Rules : 
	- On windows we can run the **netsh advfirewall show currentprofile** command (**netsh** à la base) to show current profile, then **netsh advfirewall firewall show rule name=all** to list firewall rules.
	- On Linux, we must have root perms to list firewall ules with **iptables** command. You can, however try **cat /etc/iptables**. Look also for files created by the **iptables-save** command (dumps fw configs to a file that will then be used as input for the **iptables-restore**)
- Enumerating Scheduled Tasks : 
	- On Windows, you can create and view scheduled tasks with the **schtasks /query /fo LIST /v** command (**schtasks** à la base).  
	- On Linux, you can find *cron jobs* listed under **/etc/cron.*** directories. (**ls -lah /etc/cron.`*`**).  Pay special attention to carefully inspect tasks in the **/etc/crontab** file, as those often have SysAd tasks that may be misconfigured.
- Enumerating Installed #apps and patch levels 
	- On Windows use **wmic** (*Windows Management Instrumentation*) to automate this enumeration for apps that are installed by Windows installer (**wmic product get name, version, vendor**) or querying system-wide queries (**wmic qfe get Caption, Description, HotFixID, InstalledOn**)
	- For Linux systems based on Debian, you can use the **dpkg -l** command to the same effect. 
- Enumerating #readable / #writable files and directories
	- On Windows, you can use *AccessChk* executable to automate this enumeration (**accesschk.exe -uws "Everyone" "C:`\Program Files"`**). If we can't transfer/execute arbitrary binaries, we can also try using PowerShell.   ![[Pasted image 20221028114616.png]] 
	- On Linux, we can use **find** with some refinements to do the same task.  (ex: **find / -writable -type d 2>/dev/null**)
- Enumerating #unmounted disks
	- on Windows, you can use **mountvol** to list all drives that are #mounted ***and*** physically connected but not mounted. 
	- On linux, we can use **mount** to list all mounted filesystems.  We can also read the **/etc/fstab** file, which lists all drives mounted at boot. We can use **lsblk** to view all available disks.
- Enumerating device drivers and kernel modules
	- On windows, we can use the **driverquery** command.  To better filter the results, run it within a PowerShell session. ![[Pasted image 20221028115904.png]] To get the versions of these drivers, you'll need to use the **Get-WmiObject** cmdlet ![[Pasted image 20221028120035.png]] 
	- On Linux, enumerate loaded kernel modules using **lsmod** with no addtl options.  If you want to learn more about a specific module, you can use the **/sbin/modinfo < `moduleName`>** command.
- Enumerating binaries that #AutoElevate
	- on Windows, check the status of the *AlwaysInstallElevated* registry.  If enabled (set to 1) in either *HKEY_CURRENT_USER* or *HKEY_LOCAL_MACHINE*, any user can run Windows installer packages with elevated privileges.  Check these settings using **reg query** : ![[Pasted image 20221028131243.png]] 
	- On Linux, we can look for *SUID* files using **find / -perm -u=s -type f 2</dev/null** 

## Automated Enumeration
There are various scripts available to automate the enumeration process.  One such script for Windows is *windows-privesc-check* from the Windows Privesc Check Github repo. We can also use *unix_privesc_check* for UNIX derivatives such as Linux. 

# Windows privesc examples 
- Windows #privileges and integrity.  Windows uses *access tokens* to make privileges effective.  These #tokens are made unique by using a *Security Identifier* ( #SID).  Windows also integrates an *integrity mechanism* in addition to privileges. To se integrity levels in action, use **whoami /groups** 
	- Assigns *integrity levels* to app processes and securiable objects.  4 primary #integrityLevels : 
		1. System integrity process : #SYSTEM rights
		2. High integrity process : #Administrative rights
		3. Medium integrity process : standard user rights
		4. Low integrity process : very restricted rights often used in sandboxed processes. 
- User Account Control ( #UAC).  No longer considered a security boundary by MSFT.  UAC forces applicaitons and tasks to run in the context of a non-administrative account until an admin authorizes elevated access. 2 modes : *credential prompt* and *consent prompt*. 
	- Applications with high integrity, such as *fodhelper* are ways to defeat UAC. To inspect the *application manifest* (how to handle the program when it's started), use the **sigcheck** utility (**sigcheck.exe -a -m `C:\Windows\System32\fodhelper.exe`**).
	- You can then use *Process Monitor* from Sysinternals suite to get more info about the tool as it executes using **procmon.exe**.  By searching for "Reg", we get results for apps that interact with the registry.  Rerun it using the "NAME NOT FOUND" filter to find an app interactin with a registry that doesn't exist.  Focus on the "Path contains HKCU" hive, which we have access read/write access to as the current user. 
- Leveraging #unquotedServicepath  
- Windows kernel vulns - The vast mamjority of exploits targeting kernel-level vulnerabilites are written in a low-level programming language such as C or C++, and therefore require compilation.

# Linux privesc examples 
- Insecure file #permissions (cron jobs, /etc/passwd, etc)
	-  To exploit write privileges to /etc/passwd by adding another superuser and corresponding password hash. : 
		1. Generate the password has using **openssl** and the **passwd** argument.  (**openssl passwd `<desiredPassword>`**)
		2. Echo the root2 user with the new pwhash to the /etc/passwd file (**echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd**)
		3. Switch user into the root2 user and enter the password (**su root2**)
		4. Validate your privs with the **id** command.