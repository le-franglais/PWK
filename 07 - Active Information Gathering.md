# DNS Enumeration
- The first server in the #DNS chain is the *DNS Recursor*. The recursor contacts one of the servers in the DNS root zone, and the root server responds with the address of the server responsible for the zone containing the *TLD* (.com, .net, .fr, .org, .edu, etc)
- Once the recursor has the #TLD DNS server, it queries it fo rthe address of the authoritative NS for the domain.  This is the final step and contains the DNS records in a local database known as the ***zone file***.
	- Contains 2 zones : the *forward lookup zone* (used to find the IP of a specific hostname), and the *reverse lookup zone* (if configured by the admin, used ot find the hostname of a specific IP addy)

## Interacting with the DNS Server
Different records are possible for each domain, and can be enumerated using the **host -t** command (**-t** specifies the type of record)
- NS - #Nameserver.  Contain the name of the authoritative servers hosting the DNS records for a domain
- A - IPv4 address of a hostname
- AAAA - IPv6 address of a hostname
- MX - Mail exchange - names of servers responsible for handling email for the domain
- PTR - pointer records used in reverse lookup zones 
- CNAME - Canonical Name Records used to create aliases for other host records
- TXT- text records contianing arbitrary data
You can also use the **host** command to automate forward looksups in a bash script
![[Pasted image 20221021173113.png]]

If the DNS admin configured PTR records for the domain, we could scan teh approx range with reverse lookups to request the hostname for each IP;

## #zoneTransfers
This s a *database replication* between related DNS servers in which the *zone file* is copied from a master DNS server to a slave server
- should only be allowed to authorized slave servers, but often misconfigured
- **host -l < domain name > < dns server addy >**

## DNS Tools
- #DNSRecon
- #DNSenum

# Port Scanning
_Note_ : Port scanning is not representative of traditional user activity and could be considered illegal in some jurisdictions.

It's important to not ethat port scanning generates network traffic, which could subsequently be "seen" by network teams or defense tools.  This could also have **real** impacts on network performance.

## TCP/UDP Scanning
- Connect scanning is a **TCP** port scanning technique that relies on the ***complete*** 3-way handshake to determine if a port is open
- A **UDP** port scan is different in that it's a connectionless protocol.  As a result, an *ICMP port unreachable* message indicates that the port is closed.  This is, however, ***unreliable*** when a firewall is filtering the port (in this case, the lack of an ICMP message will cause the scanner to report *open*)

## Nmap
Scans the top 1000 ports by default -- a full scan of a Class C network would send over 1gb of traffic

### Stealth/SYN scanning (**-sS**)
Sends a SYN packet and looks for the SYN/ACK reply ; does ***NOT*** complete the handshake (will not appear in application logs).  Of note, modern firewalls ***WILL*** note the incomplete handshake

### Connect Scanning (**-sT**)
Much longer since the connect scan must wait for the connection to complete before returning a port status

### UDP scanning (**-sU**)
Uses the *ICMP* method, combined with sending *protocol-specific* packets for common ports.  Requires the use of **sudo**, and can be used in conjunction with a SYN Scan to build a complete target picture.

### Network sweeping (**-sn**)
Use this option in a *grep-able* (**-oG**) format to determine live hosts on a network
- **grep Up scanname.txt | cut -d " " -f 2**

### Banner-grabbing/Service enumeration
Examine banners (**-sV**) and enumerate services with specialized scripts (**-A**) to ID services

### NSE
scripts are located in **/usr/share/nmap/scripts** 

## Masscan
Very fast, can be less accurate.  Must be installed using **apt install masscan**. Meant to scan the internet

## SMB Enumeration
### #NetBIOS
Listens on TCP/139 and several UDP ports.  Often enabled together with #SMB for backwards capability (NetBIOS over TCP, NBT).
- Can be scanned with Nmap and with NSE scripts, but can *also* be scanned with **nbtscan**, a command-line tool
	- **sudo nbtscan -r 10.11.1.0/24**

## NFS Enumeration
### Scanning for NFS shares
Portmapper and RPCbind both run on TCP/111. RPCbind is mapped to 111, but the redirection is often to TCP/2049.

When a directory is being shared (such as the entire /home directory), we can access it by mounting it on our machine and disabling file locking.
- **mkdir < directoryName >**
- **sudo mount -o nolock 10.11.1.72:/home ~/home/**
- **cd home/ && ls -lah**

## SMTP Enumeration
Key comands include ***VRFY*** (verify an email address) and ***EXPN*** (asks the server for membership of a mailing list)

## SNMP Enumeration
- Use a tool such as **onesixtyone** to attempt a brute force attack against a list of IPs.
- **snmpwalk** allows you to query snmp values provided you know the SNMP read-only community string ("public" in most cases)
	- Using **snmpwalk** you can enumerate the whole MIB tree, windows users, windows running processes, open TCP ports, or installed software.


