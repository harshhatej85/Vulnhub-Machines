# Machine Details

#### Machine Link
https://www.vulnhub.com/entry/fristileaks-13,133/
#### OS:
Linux (CentOS)
#### Web-Technology
PHP/5.3.3
#### IP
192.168.56.5
#### Users:
eezeepz
#### CREDENTIAL (ANY):
eezeepz : keKkeKKeKKeKkEkkEk
admin : thisisalsopw123
frisitgod : LetThereBeFristi!

MySQL credentials :
eezeepz : 4ll3maal12

TP -> Directory/Files Bruteforcing -> Nikto -> Manual Inspection

----

# Nmap Results:

```bash
nmap -v -sV -sT -sC -A 192.168.56.5 --open

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
| http-robots.txt: 3 disallowed entries 
|_/cola /sisi /beer
```


---

# Web Service Enumeration

Upon Using Directory and File bruteforcing we did not find anything interesting just two images.
<img width="1375" height="833" alt="image" src="https://github.com/user-attachments/assets/cbc7c5e5-bd52-47cd-99f9-d2627ee6a36f" />

---
## Manual Enumeration

Inside the robots.txt file at 192.168.56.5/robots.txt we found this :

```
User-agent: *
Disallow: /cola
Disallow: /sisi
Disallow: /beer
```


When visiting these directories :

<img width="502" height="524" alt="image" src="https://github.com/user-attachments/assets/6ae0885d-ebc7-4b61-94a7-d28ad2174dcc" />



Now here we observed that fristi, cola, sisi, beer are all types of drinks. So lets see if a page name fristi exists like cola, sisi and beer.
<img width="1914" height="1046" alt="image" src="https://github.com/user-attachments/assets/3505a562-5ec5-4d90-b28c-634b734473bc" />


We found the admin portal !!
<img width="1920" height="1048" alt="image" src="https://github.com/user-attachments/assets/656a2bb7-57fa-41ae-ad76-76a01ff581a9" />


In the source code we found a username and also some hints about base64 encoded image :
<img width="1919" height="1047" alt="image" src="https://github.com/user-attachments/assets/9cee3468-a05d-4f95-aece-16683aa4b1a3" />

Below in the source code we found a comment which looked like base64 encoded data, and we have a hint that images are being encoded into base64 by the developer.
<img width="1249" height="775" alt="image" src="https://github.com/user-attachments/assets/a7d31cbf-1a68-42be-9583-dca606709a1c" />

Now lets copy this base64 data and use online base64 to image converters to convert it into an image :
<img width="1350" height="660" alt="image" src="https://github.com/user-attachments/assets/3c37f2ae-97e1-4f51-8c48-c9cd08914bc2" />


In the image we found the password :
**keKkeKKeKKeKkEkkEk**

Now we got a pair of username and password →
eezeepz : keKkeKKeKKeKkEkkEk

Login with this credential on IP/fristi :
<img width="1157" height="488" alt="image" src="https://github.com/user-attachments/assets/c20a56fa-7ced-4760-a06b-1a8c99b11780" />

We found a page with image-upload functionaliy.

Here we cannot upload php files directly, it only accepts png,jpg and gif images.
<img width="973" height="563" alt="image" src="https://github.com/user-attachments/assets/3633a516-1216-482b-b048-7fa2bb2346a8" />

Now we know from the nmap scan that this web application is running on php. So lets download a php-reverse-shell.php code.
[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Edit this code with ip and port. Rename the file to php-reverse-shell.php.png to bypass the extension check on the application. Than finally upload the file
```bash
cp php-reverse-shell.php php-reverse-shell.php.png
```
After this upload the image and it will upload successfully.

Now start the nc listner on defined port.
```bash
nc -nlvp 443
```

Now go to the **/fristi/uploads/(file-name)** location, reverse shell file locaiton in my case : http://192.168.56.5/fristi/uploads/php-reverse-shell.php.png

If everything ran right, you will get the reverse shell on the nc listner :
<img width="1718" height="550" alt="image" src="https://github.com/user-attachments/assets/a32c69c2-e332-40f1-ad84-fd11dba2dd46" />

---
### Making Shell Usable

Now first we need to get a pty and make the terminal stable for that use this command :
```bash
python -c 'import pty;pty.spawn("/bin/bash")’
```

Also change the colour and behaviour of terminal :
```bash
PS1="\e[0;31m[\u@\h \W]\$ \e[m "
```

# Privilege Escalation admin user

In the /home directory we found the eezeepz directory is readable. Here we found a note.txt file :
<img width="935" height="687" alt="image" src="https://github.com/user-attachments/assets/10bd4387-3785-4faa-b650-53e2696cb79c" />


Now we know that in the /tmp location if we place a runthis named file it will execute with the power of admin user. So :

```bash
cd /tmp
echo "/home/admin/chmod 777 /home/admin/" > runthis
chmod +x runthis
```

This will change the directory permission of admin user. Once done go to the /home/admin directory.
<img width="1141" height="708" alt="image" src="https://github.com/user-attachments/assets/17a2e293-1e84-4d45-b477-879873f70d4d" />


Here we found that this python program converts a string to base43, than reverses it and finally converts it in rot13 format. So to decode this we need to do the reverse of this process.

The cryptedpass.txt file has the string.

Decode this in rot13 :
<img width="1911" height="664" alt="image" src="https://github.com/user-attachments/assets/5bc5f99b-c6d1-4f21-a864-2fa173f56b33" />

Now reverse this string and decode in base64.
```bash
rev <<< "zITM3B3bzxWYzl2cphGd" | base64 -d
thisisalsopw123
```

We found the password to be :
**thisisalsopw123**
Switch to admin user with this password :
<img width="510" height="222" alt="image" src="https://github.com/user-attachments/assets/1c6ddb44-736a-46e1-ba1c-ee54c851d88c" />

We successfully are admin user now.

---
# Privilege Escalation root user

Also we have another text file in this directory named whoisyourgodnow.txt this file also contains a simlar password, decode the password in a similar way.

```bash
cd /home/admin
cat whoisyourgodnow.txt
=RFn0AKnlMHMPIzpyuTI0ITG
```

Decode this in rot13 :
<img width="1857" height="579" alt="image" src="https://github.com/user-attachments/assets/bd179e57-4e48-4bbf-afbb-b455e738d0d7" />

```bash
rev <<< "=ESa0NXayZUZCVmclhGV0VGT" | base64 -d
LetThereBeFristi!
```

<img width="768" height="50" alt="image" src="https://github.com/user-attachments/assets/dc3f16f8-2ffc-4885-97c6-cebae69f7582" />

Now looking at the situation we have another user named fristigod and the text file name was whoisyourgodnow.txt also the decoded text is **LetThereBeFristi!** so this may be the password of **fristigod** user.

Switch to fristigod user :
```bash
su - fristigod
LetThereBeFristi!
```

<img width="691" height="302" alt="image" src="https://github.com/user-attachments/assets/adf1abb0-24d5-4961-80e8-c04b2f72ac4b" />


Now lets check for the commands allowed to fristigod 
```bash
sudo -l
```
<img width="957" height="397" alt="image" src="https://github.com/user-attachments/assets/6ee096b9-f4aa-4bcd-aa58-0fb510ab8420" />

here we found that we can use sudo on this doCom file. Important thing here is that the usernaem should be fristi.

```bash
cd /var/fristigod/.secret_admin_stuff/

sudo -u fristi ./doCom bash
```

<img width="1310" height="646" alt="image" src="https://github.com/user-attachments/assets/1cd7034e-95a7-4fce-b638-18a90986b0c5" />

---

