# Password attacks 
Take 2 primary forms : *dictionary* attacks (using #wordlists) or *brute force* attacks (every possible character in a password).  If we can get privileged access to a system and we can extract #password #hashes, then we can look at *password cracking* or using *pass-the-hash* attacks.

## Wordlists
Located in **/usr/share/wordlists/** directory. 

## creating standard wordlists 
- Use of **cewl** to help crawl a target website and take words meeting a certain criteria and adding them to a given wordlist
	- **cewl www.megacorpone.com -m 6 -w megacorp-cewl.txt** 
	- Once we've determined a potential password policy, we can use **john the ripper** to generate a custom wordlist by editing the **/etc/john/john.conf** file in the [*List.Rules:Wordlist*] segment and appending a new rule.
	- Now - mutate the wordlist. **john --wordlist=megacorp-cewl.txt --rules --stdout > mutated.txt** 

## Brute force wordlists
Another powerful password generator that comes included with Kali is **crunch**.
![[Pasted image 20221028165507.png]]
To generate a wordlist that matches requirements, we must specify a minimum and maximum word length and describe our rule pattern with **-t** (**crunch 8 8 -t ,^^@@``%%%``**)  OR using a specific combination of characters and saving them to a file. (**crunch 4 6 0123456789ABCDEF -o crunch.txt**)

## Common network service attack methods 
***Note***: password attacks against network services are noisy and can crash services. Popular services for this include **THC-Hydra**, **Medusa**, **Crowbar**, **spray** 
- HTTP htaccess attack with #Medusa : speedy, massively parallel, modular, login brute forcer 
- RDP attack with **crowbar** : Designed to leverage SSH keys rather than passwords.  *Note* : when attacking RDP wiht #Crowbar, specify one thread as RDP does not handle mutliple threads well.
- SSH Attack with #THC-Hydra : **hydra -l kali -P /usr/share/wordlists/rockyou.txt ``ssh://127.0.0.1``** (**-l** for username, **-P** for password wordlist, and **protocol://IP**).
- HTTP POST Attack with THC-Hydra : can be called using **hydra http-form-post -U** command, and then leveraging the page source of the web app to fill out the required information 

## Leveraging Password Hashes
*A useful tool that can assist with hash-type identification is **hashid*** 
- On a linux system, hashes are stored in the **/etc/shadow** file.  On Windows systems, hashes are stored in the *Security Accounts Manager* (***SAM file***). For Windows 2003 and below, 2 hashes are stored: LAN Manager ( #LM, *DES*) and NT LAN Manager ( #NTLM, *MD4*)
- *Note* - The SAM database cannot be copied while the OS is in use because the Windows Kernel keeps an exclusive file system lock on the file.  We can, howev, use *mimikatz* to mount in-memory attacks designed to dump the SAM hashes from the ***Local Security Authority Subsystem ( #LSASS)*** process memory where they are cached.
	- First, launch #mimikatz from an admin command prompt
	- Second, execute **privilege::debug** command (enables *SeDebugPrivilge*)
	- Finally, execute **token::elevate** command
	- Then, execute **lsadump::sam** 
- Passing the hash in Windows ( #PtH) - possible in Windows b/c #NTLM/LM hashes are not salted using *pth-winexe*, from the Passing-The-Hash toolkit.
	- Some paps like Windows Defender use the Web Proxy Auto-Discovery Protocol (WPAD) to detect proxy settings.   On the local network we ould poison these requests and force NetNTLM authentication with a tool like *Responder.py*