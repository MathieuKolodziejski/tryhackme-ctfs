# Room: Digdug

## General Info
- **Room link:** [Digdug](https://tryhackme.com/room/digdug)

## Description

Oooh, turns out, this 10.10.152.48 machine is also a DNS server! If we could dig into it, I am sure we could find some interesting records! But... it seems weird, this only responds to a special type of request for a givemetheflag.com domain?

## Provided Files / Links
- [x] Target Machine IP (10.10.152.48)

## Enumeration / Recon

From the description of the room, it seems like I will have to use dig on IP 10.10.152.48 and the target domain is `givemetheflag.com`.

 I first try to get any type of records `dig @10.10.152.48 givemetheflag.com` and I actually find the flag right away: `flag{0767ccd06e79853318f25aeb08ff83e2}`

It feels like it shouldn't be that easy. When looking at https://dnsdumpster.com/, there is actually a subdomain `ctf.givemetheflag.com` and I am pretty sure there is something wrong with the room.


