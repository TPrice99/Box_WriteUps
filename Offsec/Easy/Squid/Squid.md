### Enumeration
```
IP=192.168.170.189
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p135,139,445,3128,49667 -A $IP
```
### Ports 
- 135
- 139
- 445
- 3128
- 49666
- 49667
### Foothold
- 21
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
- 3128
	- Default landing page
		- ![[Pasted image 20240711090554.png]]
	- Hack tricks
		- squid is a proxy
		- python spose.py --proxy http://192.168.170.189:3128 --target $IP
			- Ports 3306 and 8080 are open
	- Versions
		- squid 4.14
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
- 8080
	- Setup proxy to $IP and 3128
	- Default
		- ![[Pasted image 20240711091748.png]]
	- Version
		- wampserver 3.2.3 - 64bit
			- sqli
		- Apache/2.4.46 (Win64) PHP/7.3.21 - Port defined for Apache
		- mysql 5.7.31 - Port defined for MySQL: 3306
		- mariadb 10.4.13 - Port defined for MariaDB: 3307
		- php 7.3.21
		- Adminer 4.7.7
	- phpmyadmin
		- Username: root
		- No password
		- Now logged in
		- Version: phpmyadmin 5.0.2
		- Use SQL command to upload shell
			- https://gist.github.com/BababaBlue/71d85a7182993f6b4728c5d6a77e669f?ref=benheater.com
			- ![[Pasted image 20240711093650.png]]
			- ![[Pasted image 20240711093658.png]]
			- This allows us to upload files
			- Upload php webshell
			- ![[Pasted image 20240711093714.png]]
			- Now able to run commands
### PE
#### Windows
- `nt authority\local service
	- whoami /all
		- SeChangeNotifyPrivilege enabled
		- SeCreateGlobalPrivilege enabled
		- `NT AUTHORITY\SERVICE`
	- Supposed to follow: https://itm4n.github.io/localservice-privileges/
		- Did not work
### Credentials
### Lessons Learned