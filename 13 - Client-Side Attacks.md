Client side attacks exploit weaknesses in the *client* software instead of exploiting *server* software.  As such, most client-side attacks *require some form of interaction with the target*.

# Understanding your target

## Passive Client Info gathering
As before, this is characterized by not actually engaging with teh target, rather by leveraging existing third party tools and resources to determine what the client infrastructure may look like.

## Active client info gathering
Encompasses techniques such as *social engineering* and *client-side attacks* to gather information by interacting in a trackable way with the target.
- Client #fingerprinting -- web browsers tend to be a good vector for this.  When crafting a link you could use the *Fingerprintjs2* JavaScript library to help with this identification ![[Pasted image 20221026100829.png]]

## Leveraging HTML applications
- If a file is created with teh extension of *.hta* instead of *.hml*, IE will automatically interpret it as a HTML Application and offer the ability to execute it using the **mshta.exe** program. it is of note here that this is an attack vector **specifically** targeting Internet Explorer.
- HTML Applications are executed outside the browser so we can use features often blocked wihtin the browser.
- To launch an HTA  attack, you can use msfvenom to create the #HTA payload based on PowerShell using the hta-psh output format 

## Exploiting Microsoft Office
- Microsoft Word #Macros --  *macros* are a series of commands and instructions that are grouped together to accomplish a task programmatically.  Manage dynamic content and link documents with external content.  *.doc* and *.docm* formats support macros, but *.docx* do not.  Can embed a PowerShell reverse shell code in the create macro functionality will launch a powershell reverse shell once the document is opened/re-opened and the macro enabled.
- Object linking and embedding (**OLE**) abuses MSFTs document embedding feature by embedding Windows Batch files (*.bat*, older format replaced by Windows-native scripting languages such as VBScript and PowerShell).  We can even change the appearance of the display icon and caption that the user will see to be more benign, such as an excel icon or something else. ![[Pasted image 20221026110304.png]]
- both of these options work well when served locally (inside the network).  However, when served from the internet we need to bypass Protected View (disables all editing and modificaitons and blocks the execution of macros or embedded objects)


