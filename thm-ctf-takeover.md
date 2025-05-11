# Room: Takeover

## General Info
- **Room link:** [Takeover](https://tryhackme.com/room/takeover)

## Description

Hello there,

I am the CEO and one of the co-founders of futurevera.thm. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.

Recently blackhat hackers approached us saying they could takeover and are asking us for a big ransom. Please help us to find what they can takeover.

Our website is located at https://futurevera.thm

## Provided Files / Links
- [x] Target Machine IP (10.10.20.120)

## Enumeration / Recon

As mentionned in the description of the room (`This challenge revolves around subdomain enumeration.`), I'll be using ffuf to fuzz the host (I filtered out size 4605 in the results):

```
ffuf -H  "Host : FUZZ.futurevera.thm" -u https://10.10.20.120 -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fs 4605
```

I found 2 subdomains:

```
support                 [Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 19ms]
blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 23ms]
```
## Websites analysis

I first added the new subdomains to the /etc/hosts files.

I accessed the blog.futurevera.thm website but didn't find any relevant information.

I then accessed the support.futurevera.thm website and in the certificates, I noticed that there was another subdomain:

`DNS Name   secrethelpdesk934752.support.futurevera.thm`

I added this subdomain in the /etc/hosts file, I accessed the page through https but this page is actually a copy of futurevera.thm

I then tried to access this page through http and found the flag **flag{beea0d6edfcee06a59b83fb50ae81b2f}**.

`We can’t connect to the server at flag{beea0d6edfcee06a59b83fb50ae81b2f}.s3-website-us-west-3.amazonaws.com.`