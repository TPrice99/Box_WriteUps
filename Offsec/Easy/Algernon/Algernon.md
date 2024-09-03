### Enumeration
```
IP=192.168.187.65
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p21,80,135,139,445,5040,7680,9998,17001,49665,49666,49667,49668,49669 -A $IP
```
#### Ports 
- 21
	- Microsoft ftpd
- 80
	- Microsoft IIS httpd 10.0
- 135
	- Microsoft Windows RPC
- 139
	- Microsoft Windows netbios-ssn
- 445
	- microsoft-ds
- 5040
	- unknown
- 7680
	- tcpwrapped
- 9998
	- Microsoft IIS httpd 10.0
- 17001
	- MS .NET Remoting services
- 49665
	- Microsoft Windows RPC
- 49666
	- Microsoft Windows RPC
- 49667
	- Microsoft Windows RPC
- 49668
	- Microsoft Windows RPC
- 49669
	- Microsoft Windows RPC
- OS: Windows XP?
### Foothold
- 21
	- ftp anonymous@$IP
		- Successful
		- Microsoft FTP Service
		- /imapretreival
			- empty
		- /logs
			- 2020.05.12-administrative.log
				- admin user for webmail
		- /popretreival
			- empty
		- /spool
			- empty
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
		- ftp:ftp
		- anonymous:amonymous
		- ftp:b1uRR3
- 445
	- smbclient -N -L //$IP
		- Access denied
	- nmap --script smb-vuln* -p139,445 $IP
		- nothing
- 80
	- Default:
		- ![[Pasted image 20240628090044.png]]
	- dirsearch -u http://$IP
- 9998
	- ![[Pasted image 20240628090104.png]]
	- Googled smartmail exploit
		- exploit-db 49216
		- Edited IP and port. Ran script. Got shell as nt auth system
### Lessons Learned
- Check every port for a version. Google the version and look for exploits. Try the found exploits and see if they work