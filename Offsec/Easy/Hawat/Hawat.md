### Enumeration
```
IP=192.168.170.147
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,17445,30455,50080 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.4 (protocol 2.0)
- 17445
	- unknown
	- http
- 30455
	- nginx 1.18.0
- 50080
	- Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
### Foothold
- 17445
	- Default landing page
		- ![[Pasted image 20240715112402.png]]
	- Login
		- ![[Pasted image 20240715112517.png]]
	- Register
		- allow to create account
	- Logged in
		- /user
			- clinton
			- dummy
			- Allowed to edit the password
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 30455
	- Default landing page
		- ![[Pasted image 20240715112411.png]]
	- Source code
		- `<!-- Test adds with URL/?title=test -->
			- ![[Pasted image 20240715113251.png]]
			- Vulnerable to XSS
				- `<script>print()</script>`
	- Directory Brute force
		- dirsearch -u http://$IP
			- /phpinfo.php
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 50080
	- Default landing page
		- ![[Pasted image 20240715112426.png]]
		- ![[Pasted image 20240715112438.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- /cloud
				- admin:admin
				- ![[Pasted image 20240715114412.png]]
				- Download issuetracker.zip
					- IssueController.java
						- issue_user:ManagementInsideOld797
			- /images
			- /4
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
### PE
#### Linux
- USERNAME
	- id
	- sudo -l
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- Users with console
	- linpeas
#### Windows
- USERNAME
	- whoami /all
	- net user USERNAME
		- Check group memberships
	- systeminfo
		- `./wes.py ~/OSCP/boxes/medjed/systeminfo.txt  > ~/OSCP/boxes/medjed/systeminfo_exploits.txt`
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
	- Users with console
	- Unquoted
		- cmd
			- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
		- PS1
			- Get-CimInstance -ClassName win32_service | Select Name,State,PathName
		- icacls Filepath before .exe
			- Looking for W or F
	- mimkatz
		- mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
		- mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"
### Credentials
### Lessons Learned