### Enumeration
```
IP=192.168.209.242
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
### Ports 
- 22
- 80
### Foothold
- 80
	- ![[Pasted image 20240705104939.png]]
	- Login
		- admin:admin
			- ![[Pasted image 20240705105022.png]]
	- Directory Brute force
		- dirsearch -u http://$IP
			- /ajax
				- black page
			- /babel.config.js
				- babel plugin
			- /bin
				- console file
					- ![[Pasted image 20240705105350.png]]
					- maybe useful to run console commands
			- /CHANGELOG.md
				- 10.0.2
				- glpi 10.0.2 exploit
					- Unauthenticated RCE
					- http://192.168.209.242/vendor/htmlawed/htmlawed/htmLawedTest.php
					- takes alot of playing with variables to get payload to work
			- /config
				- glpicrypt.key
				- config_db.php
					- Loasd to white page
			- /files
				- alot of sub folders
			- /inc
				- /blank page
			- /install
				- blank page
			- /INSTALL.md
				- nothing useful
			- /lib
				- blank page
			- /php.info
				- PHP Version 7.4.3-4ubuntu2.17
			- /templates
			- /status.php
				- ![[Pasted image 20240705105935.png]]
			- /src
			- /vendor
				- http://192.168.209.242/vendor/glpi-project/inventory_format/
					- A bunch of directories that might be useful
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
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