The goal of this section is to manipulate the directional flow of targeted traffic.
*Tunneling* : involves encapsulated a protocol w/in a different protocol.

# Port Forwarding
Redirecting traffic destined for one IP address and port to another.

*rinetd* is a #portForwarding tool to redirect traffic, considered easy to configure and available in the Kali repos.  **apt update && apt install rinetd**.
- modify the **/etc/rinetd.conf** file to input the required parameters (bindaddress and bindport (the "listening" IP addy) and the connectaddress and connectport (the destination port and addy.))
- give it a **sudo service rinetd restart** 

# SSH Tunneling

## SSH #localPortForwarding 
Similar to rinetd #portForwarding, allows us to tunnel a local port to  a remote server using SSH as the transport protocol.  Invoked using **ssh -L** command. This technique is limited to a particular IP address and port. Syntax :
- **ssh -N -L [bind_address:]port:host:hostport [username@address]** ![[Pasted image 20221102120232.png]]
- In short, we will set up port forwarding (**-L**) with #bindport 445 on our local machine (**0.0.0.0:445:**) to a port on another host/server (**192.168.1.110:445**), all though a session to the #pivot machine (**student@10.11.0.128**).  ![[Pasted image 20221102120814.png]] 
- *Note : Samba needs to be configured with a minmum SMB version of SMBv2*.

## SSH #remotePortForwarding
Can be thought of as the reverse of a local port forwarding in that a port is opened on the *remote* side of the connection and traffic sent to that port is forwarded to a port on our local machine (the machine initiating the SSH client). Invoked using **ssh -R** command.  Syntax  :
- **ssh -N -R [bind_address:]port:host:hostport [username@address]** ![[Pasted image 20221103093908.png]]
- In short, we will **ssh** out to our Kali machine as the user **kali**, and a remote forward through a listener on localport **2221**, then foward connections to the internal Linux machine's **TCP/3306**.  ![[Pasted image 20221103094004.png]] 

## SSH #dynamicPortForwarding
Allows us to set a local listening port and hive it tunnel incoming traffic to any remote destination through the use of a proxy. Useful when compromising a machine connected to a current network that has an additional network interface connected to a *different* network. Dynamic Port Forwarding allows us to target additional ports on the different network without having to establish different tunnels for each port or host of interest. Invoked using **ssh -D**.  Syntax : 
- **ssh -N -D [address to bind to]:[port to bind to]  [username@sshServerAddress]** ![[Pasted image 20221103095035.png]] 
- In short, we've created a local **SOCKS4** application proxy (**-N -D**) on our kali linux terminal TCP/8080 (**127.0.0.1:8080**) which will tunnel all incoming traffic to any host in the target network through the compromised machine.  ![[Pasted image 20221103095346.png]] 
- We can run any network application through HTTP, SOCKS4, and SOCKS5 with the help of #proxyChains  
	- Configuration happens in teh main config file (**/etc/proxychains.conf**) by adding the SOCKS4 proxy to it. ![[Pasted image 20221103095627.png]] 
	- To run tools through a #SOCKS4 proxy, prepend each command with **proxychains** 

## PLINK.exe 
**plink.exe** is a Windows-based command line SSH client (part fo the PuTTY project). Invoked using the **plink.exe** command.
- Can be used to connect via **-ssh** to our attacking machine (**10.11.0.4**) as the user (**-l kali**) with a given password (**-pw ilak**) to create a remote port forward (**-R**) of port 1234 (**10.11.0.4:1234**) to the #MySQL port on the Windows target (**127.0.0.1:3306**) ![[Pasted image 20221103100729.png]] 
- you should pipe the answer to the prompt with the **cmd.exe /c echo y** command : ![[Pasted image 20221103100944.png]]

## #netsh
The use of this utility comes into play when you have *SYSTEM-level* shell on your pivot machine (b/c of this privilege level we do not have to deal with UAC)
- the Windows pivot system must have *IP Helper* service running **and** *IPv6* support must be enabled for the interface we want to use.
- For this, we will use the **netsh** (**interface**) context to **add** an IPv4-to-IPv4 (**v4tov4**) proxy (**portproxy**) listening on 10.11.0.22 (**listenaddress=10.11.0.22**), port 4455 (**listenport=4455**) that will forward to the Windows 2016 Server (**connectaddress=192.168.1.110**) on port 445 (**connectport=445**) ![[Pasted image 20221103113157.png]] 
- b/c we have #SYSTEM privs, we can easily address the fact that Windows Firewall disallows inbound connections on TCP/4455 by adding a firewall rule to allow inbound connections on that port.
	- **netsh advfirewall firewall add rule name="forward_port_rule" protocol=TCP dir=in localip=10.11.0.22 localport=4455 action=allow** 

## #HTTPTunneling through deep packet inspection
With deep packet inspection, devices may only allow specific protocols.  In this case, previous tunnels relying on SSH would fail.  In this case, the goal is to initiate an RDP connection from our kali mahcine to the windows Server 2016 through the compromised Linux server using only the HTTP protocol.
- **HTTPTunnel** and **stunnel** can encapsulate our traffic within HTTP requests using a client/server model
- The *input* of this tunnel will be on our Kali machine (**localhost:8080**) and the tunnel will *output* to the compromised Linux machine on listening port 1234 (across the firewall) ![[Pasted image 20221103115329.png]] 
- Start by creating a local forward from the compromised Linux machine and forwarding all requests on **0.0.0.0:8888** to the Windows Server's RDP port (**192.168.1.110:3389**) ![[Pasted image 20221103115601.png]]
- Next, create an HTTPTunnel from the compromised Linux machine to our Kali box using both client (**htc**, listen on localhost:8080, encapsulate in HTTP, and forward it across the firewall to our listening HTTPTunnel server) and server (**hts**, listen on localhost:1234 and redirect to localport:8080)
	- **hts --forward-port localhost:8888 1234**
	- **htc --forward-port 8080 10.11.0.128:1234**