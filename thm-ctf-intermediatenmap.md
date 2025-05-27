# Room: Intermediate NMAP

## General Info
- **Room link:** [Intermediate NMAP](https://tryhackme.com/room/intermediatenmap)

## Description

You've learned some great nmap skills! Now can you combine that with other skills with netcat and protocols, to log in to this machine and find the flag? This VM 10.10.38.83 is listening on a high port, and if you connect to it it may give you some information you can use to connect to a lower port commonly used for remote access!

## Provided Files / Links
- [x] Target Machine IP (10.10.38.83)


## Enumeration / Recon
- Ports Enumeration

Basic port enumeration:
`nmap -sV 10.10.38.83 -p- -T5`

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
2222/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
31337/tcp open  Elite?
```

When using netcat on port 31337, I get the following:

`nc 10.10.38.83 31337`

```
In case I forget - user:pass
ubuntu:Dafdas!!/str0ng
```

I then try this username:password pair on the first ssh port (22):

`ssh ubuntu@10.10.38.83 -p 22`

And I'm in the remote machine.

I cannot find the flag in ubuntu's home directory and find it in user's directory: 
**flag{251f309497a18888dde5222761ea88e4}**.