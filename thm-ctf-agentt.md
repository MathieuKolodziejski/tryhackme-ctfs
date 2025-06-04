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
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)

Only port 80 seems to be open.
```

- Directories Enumeration

`gobuster dir -u 10.10.222.78 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100 --exclude-length 42131`

I excluded length 42131 because I had this error:
```
Error: the server returns a status code that matches the provided options for non existing urls. http://10.10.222.78/09a67af3-699d-4b6d-b919-dadc3f46b70f => 200 (Length: 42131). To continue please exclude the status code or the length
```

After naviguating to different pages, I noticed that I was redirected to different html pages.

I changed the gobuster command to this one to only check for html pages:
```
gobuster dir -u 10.10.222.78 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -x html -t 100 --exclude-length 42131
```

But I didn't find anything interesting.

I looked for a CVE on the PHP version of the server (PHP 8.1.0-dev) and I found this CVE: https://www.exploit-db.com/exploits/49933.

I downloaded the python script, ran it and now I am root.

I look for text files in the server: `find / -name *.txt` and find the flag.txt file.
The flag is **flag{4127d0530abf16d6d23973e3df8dbecb}**.
