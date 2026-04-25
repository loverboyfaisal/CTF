#[TryHackMe:Relevant](https://tryhackme.com/room/relevant)
# Enumeration
Established nmap scan to do port scan on target to detect services and version of services
```
root~# sudo nmap -p- -sC -sV -Pn -oN 10.128.171.113 10.128.171.113
Not shown: 65527 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2026-04-25T06:57:23+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2026-04-24T06:50:01
|_Not valid after:  2026-10-24T06:50:01
|_ssl-date: 2026-04-25T06:58:03+00:00; 0s from scanner time.
49663/tcp open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m59s, deviation: 3h07m50s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-04-24T23:57:24-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-04-25T06:57:27
|_  start_date: 2026-04-25T06:50:01
```
This scan gives us a lot I will break it one by one . First , there is `netbios-ssn` exposing SMB shared file we can investigate it via *smbclient* .
```
root@# smbclient -L //10.128.171.113 --no-pass

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk      
SMB1 disabled -- no workgroup available
```
Connecting to `nt4wrksv` shared file 
```
root@# smbclient //10.128.171.113/nt4wrksv/
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 22:46:04 2020
  ..                                  D        0  Sat Jul 25 22:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 16:15:33 2020
```
Found file called password 
![image](../../images/rel-1_clean.png)
Obviously it's base64 encoded text we will decode easily.
![image](../../images/rel-2_clean.png)
Bingo! Decoding the another text we can find that we have to users Bob and Bill each with his own password.

doing directory enumeration on HTTP server port `tcp/49663` we found that we can access file password .
![image](../../images/rel-3_clean.png)
                 http://10.128.171.113:49663/nt4wrksv/passwords.txt
This lead us to that . we can upload file inside SMB shared file and request it , Sure we will upload shell but we should know precisely which backend language used. Using *wapplayzer* 
![image](../../images/rel-4_clean.png)
# Weaponization & initial access 
Using [reverse shell](https://raw.githubusercontent.com/borjmz/aspx-reverse-shell/master/shell.aspx) the upload file via *smbclient* to public shared folder. 
![image](../../images/rel-5_clean.png)
then request it via browser our curl `http://10.128.142.208:49663/nt4wrksv/service.aspx`
THEN BINGO!
![image](../../images/rel-6_clean.png)
**User Flag**
![image](../../images/rel-7_clean.png)
# Privilege escalation
## Enumeration
Going to writable file which is `C:\Windows\Temp` downloading [winPEAS](https://raw.githubusercontent.com/peass-ng/PEASS-ng/refs/heads/master/winPEAS/winPEASps1/winPEAS.ps1) script on target machine using simple python3 HTTP server to enumeration and running it.
![image](../../images/rel-8_clean.png)
This is piece of cake for privilege escalation. Searching for exploits we found this [Exploit](https://github.com/itm4n/PrintSpoofer) for it.
**Exploit it**
```
exp.exe -i -c cmd
```
Bingo!
![image](../../images/rel-9_clean.png)
**We got root flag**
![image](../../images/rel-10_clean.png)
