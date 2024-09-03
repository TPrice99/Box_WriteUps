### Enumeration
```
IP=192.168.214.52
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,22,80,8080 -A $IP
```
### Ports 
- 21
	- vsftpd 3.0.3
- 22
	- OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.18 ((Ubuntu))
- 3305
	- Apache httpd 2.4.18 ((Ubuntu))
- 8080
	- Apache httpd 2.4.18 ((Ubuntu))
### Foothold
- 21
	- ftp anonymous@$IP
		- login failed
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- creds not found
- 80
	- Default landing page
		- ![[Pasted image 20240709133824.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- `/images`
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
- 8080
	- Default landing page
		- ![[Pasted image 20240709133833.png]]
	- Versions
		- Tomcat 9?
	- Directory Brute force
		- dirsearch -u http://$IP:8080
			- `/WEB-INF/  /favicon.ico`
			- /WEB-INF/
				- web.xml
					- forbidden
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
			- zm
				- ![[Pasted image 20240709135143.png]]
				- Version
					- ZoneMinder Console v1.29.0
						- sqli
							- https://www.exploit-db.com/exploits/41239
							- We can put a php webshell
							- ![[Pasted image 20240709143152.png]]
							- Now we can run commands
							- Used busy box rev shell over port 21
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- requires password
	- uname -a
		- `Linux pebbles 4.4.0-184-generic #214-Ubuntu SMP Thu Jun 4 10:14:11 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
		- nothing interesting
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing interesting
	- Users with console
		- sally
	- linpeas
		- possible exploits
			- dirty cow
			- pwnkit
				- moved PwnKit to host
				- ./PwnKit
				- now rooted
		- /usr/share/zoneminder/www/api/app/Config/database.php
			- db: zm
			- user: zmuser
			- password: zmpass
		- /etc/zm/zm.conf
			- user: root
			- pass: ShinyLucentMarker361
	- mysql -u root -p
		- ShinyLucentMarker361
			- Successful
		- dbs
			- zm
				- Users
					- admin:`*4ACFE3202A5FF5CF467898FC58AAB1D615029441
					- crackstaion - admin
			- sys
			- mysql
			- information_schema
			- performance_schema
### Credentials
### Lessons Learned