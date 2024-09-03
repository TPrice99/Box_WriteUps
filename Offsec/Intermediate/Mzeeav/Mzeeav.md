### Enumeration
```
IP=192.168.165.33
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
- 80
### Foothold
- 21
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- RPC/ SMB
	- enum4linux $IP
- 445
	- smbclient -N -L //$IP
	- nmap --script smb-vuln* -p135,139,445 $IP
- SQL
	- Mysql
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP
	- Postgres
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt postgres://$IP
- 80
	- Default landing page
		- ![[Pasted image 20240813115609.png]]
	- Upload feature
		- ![[Pasted image 20240813115648.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- /backups/
				- backup.zip
				- /var/www/html directory
				- upload.php
					- MagicBytez MZ PEFILE
					- Find magic byte is 4D5A
						- This value is from bin2hex. We use decoded to find it equal MZ
					- Upload php webshell and add MZ to top
					- ![[Pasted image 20240814092524.png]]
					- Get RCE with busybox
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- needs password
	- uname -a
		- `Linux mzeeav 5.10.0-26-amd64 #1 SMP Debian 5.10.197-1 (2023-09-29) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- ping
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- avuser
			- Have some permissions over home
	- netstat -ano
		- Active ports
			- nothing new
	- Directories to check
		- /opt
			- Check GTFO Bins
			- fileS
				- /opt/fileS --help
					- GNU findutils
				- /opt/fileS --version
					- find (GNU findutils) 4.8.0
				- Treat as suid
					- ![[Pasted image 20240814095258.png]]
	- linpeas
		- Possible Exploits
			- dirty pipe
		- Interesting Files
	- pspy64
### Credentials
### Lessons Learned