### Enumeration
```
IP=192.168.189.45
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p80,135,139,445,3389,3573,49152,49153,49154,49155,49158,49159 -A $IP
```
### Ports 
- 80
	- GoAhead WebServer
- 135
	- RPC
- 139
	- RPC
- 445
	- Windows 7 Ultimate N 7600 microsoft-ds
- 3389
	- tcpwrapped
- 3573
	- tag-ups-1
- 49152
	- RPC
- 49153
	- RPC
- 49154
	- RPC
- 49155
	- RPC
- 49158
	- RPC
- 49159
	- RPC
- OS: Windows 7
### Foothold
- RPC/ SMB
	- enum4linux -U $IP
		- nothing, access denied
- 445
	- smbclient -N -L //$IP
		- anonymous login successful
		- no shares
	- nmap --script smb-vuln* -p135,139,445 $IP
		- smb-vuln-ms17-010: Vulnerable
		- RCE over SMBv1
		- https://github.com/3ndG4me/AutoBlue-MS17-010
			- AutoBlue-MS17-010/shellcode
				- ./shell_prep.sh
				- This builds the rev shell code
		- Tried MSF, does not work
- 80
	- ![[Pasted image 20240704091901.png]]
	- version hp power manager
		- buffer overflow via MSF or python script
		- exploit-db 10099
		- Shell code sends the rev shell back to the host lol. Use MSF  >  RCE as nt auth system
	- dirsearch -u http://$IP
		- `/cgi-bin`
### Lessons Learned
- Enumerate all ports before moving onto exploits. Exploits take time to try and troubleshoot. Move on if exploits don't work. Could be a rabbit hole