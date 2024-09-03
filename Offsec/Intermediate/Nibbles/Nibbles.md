### Enumeration
```
IP=192.168.170.47
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,22,80,5437 -A $IP
```
### Ports 
- 21
	- vsftpd 3.0.3
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 80
	- Apache httpd 2.4.38 ((Debian))
- 5437
	- PostgreSQL DB 11.3 - 11.9
		- RCE - 50847
### Foothold
- 21
	- ftp anonymous@$IP
		- failed
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- not found
- 80
	- Default landing page
		- ![[Pasted image 20240711080531.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
- 5437
	- Googled PostgreSQL DB 11.3 - 11.9
		- Exploit-db 50847 RCE
			- ![[Pasted image 20240711081050.png]]
			- This command is supposed to get you a shell. Upload a bash shell then execute with bash
### PE
#### Linux
- USERNAME
	- id
	- sudo -l
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- find
			- Use this to get root
	- Users with console
	- linpeas
### Credentials
### Lessons Learned