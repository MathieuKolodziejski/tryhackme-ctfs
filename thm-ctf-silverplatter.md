# Room: Silver Platter

## General Info
- **Room link:** [Silver Platter](https://tryhackme.com/room/silverplatter)

## Description

Think you've got what it takes to outsmart the Hack Smarter Security team? They claim to be unbeatable, and now it's your chance to prove them wrong. Dive into their web server, find the hidden flags, and show the world your elite hacking skills. Good luck, and may the best hacker win!
But beware, this won't be a walk in the digital park. Hack Smarter Security has fortified the server against common attacks and their password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist - that's how 'cool' they are). The hacking gauntlet has been thrown, and it's time to elevate your game. Remember, only the most ingenious will rise to the top. 

May your code be swift, your exploits flawless, and victory yours!

## Provided Files / Links
- [x] Target Machine IP (10.10.116.253) => created host silverplatter.thm

## Enumeration / Recon
- Ports Enumeration

`nmap -sV silverplatter.thm -O`

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-05-09 10:30 EDT
Nmap scan report for silverplatter.thm (10.10.116.253)
Host is up (0.025s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
8080/tcp open  http-proxy
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.94SVN%I=7%D=5/9%Time=681E117E%P=x86_64-pc-linux-gnu%r(
SF:GetRequest,C9,"HTTP/1\.1\x20404\x20Not\x20Found\r\nConnection:\x20close
SF:\r\nContent-Length:\x2074\r\nContent-Type:\x20text/html\r\nDate:\x20Fri
SF:,\x2009\x20May\x202025\x2014:30:22\x20GMT\r\n\r\n<html><head><title>Err
SF:or</title></head><body>404\x20-\x20Not\x20Found</body></html>")%r(HTTPO
SF:ptions,C9,"HTTP/1\.1\x20404\x20Not\x20Found\r\nConnection:\x20close\r\n
SF:Content-Length:\x2074\r\nContent-Type:\x20text/html\r\nDate:\x20Fri,\x2
SF:009\x20May\x202025\x2014:30:22\x20GMT\r\n\r\n<html><head><title>Error</
SF:title></head><body>404\x20-\x20Not\x20Found</body></html>")%r(RTSPReque
SF:st,42,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Length:\x200\r\nCo
SF:nnection:\x20close\r\n\r\n")%r(FourOhFourRequest,C9,"HTTP/1\.1\x20404\x
SF:20Not\x20Found\r\nConnection:\x20close\r\nContent-Length:\x2074\r\nCont
SF:ent-Type:\x20text/html\r\nDate:\x20Fri,\x2009\x20May\x202025\x2014:30:2
SF:2\x20GMT\r\n\r\n<html><head><title>Error</title></head><body>404\x20-\x
SF:20Not\x20Found</body></html>")%r(Socks5,42,"HTTP/1\.1\x20400\x20Bad\x20
SF:Request\r\nContent-Length:\x200\r\nConnection:\x20close\r\n\r\n");
OS fingerprint not ideal because: maxTimingRatio (1.508000e+00) is greater than 1.4
No OS matches for host
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

  - Notes:
    - ssh is open
    - http is open
    - http-proxy is open
    - No OS matches for host

- Website visit

Website seems to be built with https://html5up.net/
Found a username in the Contact page: `If you'd like to get in touch with us, please reach out to our project manager on Silverpeas. His username is "scr1ptkiddy".`

There are several sections in the HTML that are loaded at launch that I cannot find on the rendered page. One of the element is an HTML form:

```
<h3 class="major">Form</h3>
                  <form method="post" action="#">
                    <div class="fields">
                      <div class="field half">
                        <label for="demo-name">Name</label>
                        <input type="text" name="demo-name" id="demo-name" value="" placeholder="Jane Doe" />
                      </div>
                      <div class="field half">
                        <label for="demo-email">Email</label>
                        <input type="email" name="demo-email" id="demo-email" value="" placeholder="jane@untitled.tld" />
                      </div>
                      <div class="field">
                        <label for="demo-category">Category</label>
                        <select name="demo-category" id="demo-category">
                          <option value="">-</option>
                          <option value="1">Manufacturing</option>
                          <option value="1">Shipping</option>
                          <option value="1">Administration</option>
                          <option value="1">Human Resources</option>
                        </select>
                      </div>
                      <div class="field half">
                        <input type="radio" id="demo-priority-low" name="demo-priority" checked>
                        <label for="demo-priority-low">Low</label>
                      </div>
                      <div class="field half">
                        <input type="radio" id="demo-priority-high" name="demo-priority">
                        <label for="demo-priority-high">High</label>
                      </div>
                      <div class="field half">
                        <input type="checkbox" id="demo-copy" name="demo-copy">
                        <label for="demo-copy">Email me a copy</label>
                      </div>
                      <div class="field half">
                        <input type="checkbox" id="demo-human" name="demo-human" checked>
                        <label for="demo-human">Not a robot</label>
                      </div>
                      <div class="field">
                        <label for="demo-message">Message</label>
                        <textarea name="demo-message" id="demo-message" placeholder="Enter your message" rows="6"></textarea>
                      </div>
                    </div>
                    <ul class="actions">
                      <li><input type="submit" value="Send Message" class="primary" /></li>
                      <li><input type="reset" value="Reset" /></li>
                    </ul>
                  </form>
```


- Directories Enumeration

`gobuster dir -u http://10.10.116.253 -q -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

 - /images
 - /assets

Nothing found in these directories

- Directories Enumeration with proxies

`gobuster dir -u http://10.10.116.253:8080 -q -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`
 
 - /website
 - /console

No luck with those (forbidden).

After spending too much time trying to exploit ssh, I went back to the message on the contact page and tried my luck on silverplatter.thm:8080/silverpeas and it actually worked (I should have googled "silverpeas earlier").

## Exploitation / Analysis

### Silverpeas

About Silverpeas: Silverpeasis a Collaborative and Social-Networking Portal built to facilitate and to leverage the collaboration, the knowledge-sharing and the feedback of persons, teams and organizations.

CVE-2024-36042
--------------

At the login page, I found that the platform is vulnerable to a trivial authentication bypass (https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d)
Basically, I just need to remove the password field in the authentification request:

`Login=scr1ptkiddy&Password=SilverAdmin&DomainId=0` => `Login=scr1ptkiddy&DomainId=0`

And I am logged in the platform.

I went to the users page and notice that there were 2 other users: Manager and Administrator.

I logged out and logged back in with the Manager (used the same CVE as before).

In the Notification section, I found the following message :
```
Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. 

Username: tim

Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```
### Target Machine access

I connected to the target machine with ssh (with tim and the password found above).
The first flag was right in Tim's folder (in user.txt): **THM{c4ca4238a0b923820dcc509a6f75849b}**.
To find exploit in privilege escalation, I am running `linpeas.sh` in the /tmp folder (I created a webserver in my Dowloads file to send it to the remote /tmp folder).

I found something interesting in the 'Checking Pkexec policy' section:
`AdminIdentities=unix-group:sudo;unix-group:admin`

If the user is in an admin group, he can run privileged actions.
The `id` commands show that the user is in an admin group `uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)`.

I then ended up spending too much time trying this CVE (https://github.com/PwnFunction/CVE-2021-4034) but it was not actually working.
During this step, I noticed that tyler user has root privileges. The goal is now to find tyler's password to get root access.

Since Tim is in the admin group, he also has access to all the logs. I started looking in the logs for passwords (and had to exclude linpeas in my search because too many logs were generated by the linpeas script).


`grep -REi 'password' /var/log/* | grep -vi "linpeas"`:


 `8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1`

 I tried the DB_PASSWORD to see if it was reused for tyler (`su tyler`) and it worked.
Just to be sure, I checked tyler's sudo permissions (`User tyler may run the following commands on silver-platter: (ALL : ALL) ALL`) and got root access (`sudo su`).

I then found the root flag in the root folder: **THM{098f6bcd4621d373cade4e832627b4f6}**



## Final considerations

The last part of the privilege escaltion was a bit tricky (I was pretty sure that I would get access through pkexec). Next time I will make a better judgement when I see users in the admin group.
