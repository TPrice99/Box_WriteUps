### Enumeration
```
IP=192.168.158.62
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,53,80,4505,4506,8000 -A $IP
```
#### Ports 
- 22
	- OpenSSH 7.4 (protocol 2.0)
- 53
	- NLnet Labs NSD
- 80
	- nginx 1.16.1
- 4505
	- ZeroMQ ZMTP 2.0
- 4506
	- ZeroMQ ZMTP 2.0
- 8000
	- nginx 1.16.1
- OS: Linux
### Foothold
- 80
	- Default page
		- ![[Pasted image 20240624165340.png]]
	- Admin interface
		- ![[Pasted image 20240624165404.png]]
	- dirsearch -u http://$IP
		- `/blog  /sitemap.xml`
	- /search
		- ![[Pasted image 20240624170218.png]]
		- Maybe LFI through q parameter
- 4505
	- Google - ZeroMQ ZMTP 2.0
		- exploit-db 48421 - Saltstack 3000.1 - Remote Code Execution
		- Googled Saltstack 3000.1 exploit
			- https://github.com/jasperla/CVE-2020-11651-poc
			- RCE not working
			- Able to cat /etc/shadow
			- Can upload files
				- ![[Pasted image 20240624171726.png]]
				- ![[Pasted image 20240624171738.png]]
				- ![[Pasted image 20240624171746.png]]
- 8000
	- Default
		- ![[Pasted image 20240624165520.png]]
	- Googled - wheel_async
		- Salt stack unauthenticated RCE
			- https://www.rapid7.com/db/modules/exploit/linux/http/saltstack_salt_wheel_async_rce/
### PE
#### Linux
- id
- sudo -l
- uname -a
- capabilities
- suid
#### Windows
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
### Lessons Learned
- Look at all ports and services thoroughly.
- Read the entire github before doing an attack instead of banging head against one part of attack
- Check google for all possible exploit scripts