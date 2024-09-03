### Enumeration
```
IP=192.168.158.43
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p135,139,445,3389,8080 -A $IP
```
#### Ports 
- 135
	- Microsoft Windows RPC
- 139
	- Microsoft Windows netbios-ssn
- 445
	- Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
- 3389
	- Microsoft Terminal Service
- 8080
	- Apache Tomcat/Coyote JSP engine 1.1
- OS: Windows 8 server 
### Foothold
- 445
	- smbclient -N -L //$IP
		- Anonymous login successful
		- No shares
		- Unable to connect with SMB1
	- nmap --script smb-vuln* -p139,445 $IP
		- smb-vuln-cve2009-3103 - Vulnerable
- 8080
	- Default
		- ![[Pasted image 20240624163153.png]]
	- `ManageEngine ServiceDesk Plus  |  7.6.0
		- Searchsploit
			- SQLi
			- file upload
	- Google - manage engine service desk default login
		- administrator:administrator  -  successful
	- Logged in
		- ![[Pasted image 20240624163626.png]]
		- RCE exploit
			- https://github.com/PeterSufliarsky/exploits/blob/master/CVE-2014-5301.py
			- ![[Pasted image 20240624164745.png]]
### Lessons Learned
- Check all ports before deep diving on an exploit. Take a step back and look around.
- Check documentation for default credentials
- Check google for exploits