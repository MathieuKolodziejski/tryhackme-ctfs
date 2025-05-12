# Room: Corridor

## General Info
- **Room link:** [Corridor](https://tryhackme.com/room/corridor)

## Description

You have found yourself in a strange corridor. Can you find your way back to where you came?

In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). This could help you uncover website locations you were not expected to access.

## Provided Files / Links
- [x] Target Machine IP (10.10.104.45)

## Website analysis

This website is and image: a corridor with 13 doors.
When hovering over the doors, a md5 hash is displayed.

I used crackstation.net to get the hash values.

Starting from the first door on the left, to the last door on the right, I get the following md5 hash values:

```
1st door: c4ca4238a0b923820dcc509a6f75849b => 1
2nd door: c81e728d9d4c2f636f067f89cc14862c => 2
3rd door: eccbc87e4b5ce2fe28308fd9f2a7baf3 => 3
4th door: a87ff679a2f3e71d9181a67b7542122c => 4
5th door: e4da3b7fbbce2345d7772b0674a318d5 => 5
6th door: 1679091c5a880faf6fb5e6087eb1b2dc => 6
7th door: 8f14e45fceea167a5a36dedd4bea2543 => 7
8th door: c51ce410c124a10e0db5e4b97fc2af39 => 13
9th door: c20ad4d76fe97759aa27a0c99bff6710 => 12
10th door: 6512bd43d9caa6e02c990b0a82652dca => 11
11th door: d3d9446802a44259755d38e6d163e820 => 10
12th door: 45c48cce2e2d7fbdea1afc51c7c6ad26 => 9
13th door: c9f0f895fb98ab9159f51fd0297e236d => 8
```

Let's try get the hash values for 0 and 14.

```
❯ echo -n "0" | md5sum
cfcd208495d565ef66e7dff9f98764da  -

❯ echo -n "14" | md5sum
aab3238922bcc25a6f606eb525ffdc56  -

```

When accessing 10.10.104.45/cfcd208495d565ef66e7dff9f98764da I actually get the flag **flag{2477ef02448ad9156661ac40a6b8862e}**.
