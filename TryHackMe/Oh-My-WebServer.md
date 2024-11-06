## Summary 
These box was listed under the medium difficulty in TryHackMe which makes use of a vulnerability in the Apache web server's version 2.4.49 which leads to RCE. The Intial access provides entry into a docker environment, where an attacker can escalate their privileges into root via improper capabilities granted to the python executable.
The Attacker can then import a portable nmap executable onto the docker container to find a new port which is susceptible to CVE-2021-38647 and thus gains root!


### Nmap Scan
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e0:d1:88:76:2a:93:79:d3:91:04:6d:25:16:0e:56:d4 (RSA)
|   256 91:18:5c:2c:5e:f8:99:3c:9a:1f:04:24:30:0e:aa:9b (ECDSA)
|_  256 d1:63:2a:36:dd:94:cf:3c:57:3e:8a:e8:85:00:ca:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.49 ((Unix))
|_http-server-header: Apache/2.4.49 (Unix)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Consult - Business Consultancy Agency Template | Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.84 seconds
```

### Initial Access
Using this exploit https://www.exploit-db.com/exploits/50383 the web server is susceptible to RCE
![Oh My Webserver-20241106130644909](https://github.com/user-attachments/assets/5da2b69e-8e20-44e4-9d5c-d0d4c8a08d9c)

Initial access leads to entry into a docker container
![Oh My Webserver-20241106131932916](https://github.com/user-attachments/assets/f43f6f1f-f637-4ab6-b7fb-6fa3230bb0d8)

After checking all files and not finding any entry to escalate privileges, I ran linpeas to cover all my tracks and see if I missed anything and found this
![Oh My Webserver-20241106122159659](https://github.com/user-attachments/assets/f9644e64-e75c-4a78-96fa-0dea6b8f3d70)

Searching this capability on GTFO bins shows this
![image](https://github.com/user-attachments/assets/516d5513-ae84-4331-bece-265201263d1d)

Escalating to root on the container and finding user.txt
![Oh My Webserver-20241106122418723](https://github.com/user-attachments/assets/e3b03d01-f3fd-4765-bc96-089b5e7bf23c)


### Privilege Escalation
I could not find anything substantial when escalating to root on the container, there were no scripts being ran from the host box to the container. I then download an nmap portable executable from this website https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap and then uploaded it to the box. 
After running a host discovery scan on the network subnet I found this
![image](https://github.com/user-attachments/assets/dda2fb34-dff8-4058-8a27-e75e65487911)

I then ran a simple all port scan on the new IP address and found this
![Oh My Webserver-20241106130757116](https://github.com/user-attachments/assets/37559bca-a9a7-4d91-b9be-22de3de74648)

Searching online for this port along with an exploit lead to this github page https://github.com/AlteredSecurity/CVE-2021-38647
After uploading the python exploit file to the box I then ran the exploit and picked up a reverse shell as root on the main box!
![Oh My Webserver-20241106130437271](https://github.com/user-attachments/assets/0aff87fd-eec6-441b-8a90-40accae64b30)
![Oh My Webserver-20241106130500849](https://github.com/user-attachments/assets/8e4da503-da99-4e74-ab3a-0b11b2e096f6)


### Things Learned
- In this challenge, I learned of new ways of discovering ways of escaping a docker container such as by running network scans to check for undiscovered ports 
