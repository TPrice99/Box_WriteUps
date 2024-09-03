### Enumeration
```
IP=192.168.184.181
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,3000,9090 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22 - ssh
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.4
- 3000 - ppp
	- nc -nv $IP 3000
		- unknown
- 9090 - zeus-admin
	- Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
	- nc -nv $IP 9090
		- unknown
### Foothold
- 3000
	- Default landing page
		- ![[Pasted image 20240722141217.png]]
	- Versions
		- Grafana 8.3.0 (914fcedb72)
			- Directory traversal  exploit-db 50581
				- ![[Pasted image 20240722141532.png]]
				- /var/lib/grafana/grafana.db
					- ![[Pasted image 20240722143341.png]]
					- `prometheusPrometheusserverhttp://localhost:9090sysadmin{}2022-02-04 09:19:592022-02-04 09:19:59{"basicAuthPassword":"anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w=="}HkdQ8Ganz`
					- Tried base64 -d, failed
					- Decrypt: https://github.com/jas502n/Grafana-CVE-2021-43798/blob/main/README.md
						- 
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
- 9090
	- Default landing page
		- ![[Pasted image 20240722141240.png]]
	- Versions
		- prometheus", version="go1.17.5"}  2.32.1
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
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
		- sysadmin
		- root
	- netstat -ano
		- Active ports
	- linpeas
		- Possible Exploits
		- Interesting Files
### Credentials
### Lessons Learned