---
layout: single
title: "Hack The Box Write-up #4 : Blunder"
excerpt: "Writeup for Blunder, a machine provided by HacktheBox."
date: 2020-10-17
toc: true
toc_sticky: true
header:
  tagline: "Writeup for Blunder , a machine provided by HacktheBox."
  overlay_image: /assets/images/background.jpg
  overlay_filter: 0.5
  actions:
    - label: "Learn More"
      url: "https://www.hackthebox.eu/home/machines/profile/254"
  teaser: /assets/images/blunder-writeup/blunder.png
  teaser_home_page: true
  
categories:
  - hackthebox
tags:  
  - linux
  - cve
  - bludit-cms
---

![Card](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/blunder-writeup/blunder.png)

## Summary

Blunder is an easy rated linux machine provided by hackthebox where are mainly represented with some Bludit CMS CVEs.
We use an Authentication Bypass through bruteforcing CVE to login on the admin dashboard. Later on we use yet another CVE (Arbitrary File Upload through images)
to grab a shell on the machine. A newer version of Bludit is configured in the machine where we find a hash for a system user which we crack later.
A sudo misconfiguration easily drops root shell.

## Enumeration
As always I start with a nmap scan to grab the initial information :
### nmap

```
root@kali:/home/suljot# nmap -sC -A 10.10.10.191
Starting Nmap 7.91 ( https://nmap.org ) at 2020-10-17 16:28 CEST
Nmap scan report for 10.10.10.191
Host is up (0.070s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   72.10 ms 10.10.14.1
2   72.03 ms 10.10.10.191
```
Port 21 seems closed so it obviously would be useless for now which leaves port 80 the only vector to gain more information.

### Website

![website](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/blunder-writeup/blunder-web.png)

Represented with a simple blog webpage which doesnt tell much useful information for the box.

Viewing the source of the webpage reveals things that could be useful : 

```html
<!-- Dynamic title tag -->
<title>Blunder | A blunder of interesting facts</title>

<!-- Dynamic description tag -->
<meta name="description" content="HackTheBox">

<!-- Include Favicon -->
<link rel="shortcut icon" href="http://10.10.10.191/bl-themes/blogx/img/favicon.png" type="image/png">

<!-- Include Bootstrap CSS file bootstrap.css -->
<link rel="stylesheet" type="text/css" href="http://10.10.10.191/bl-kernel/css/bootstrap.min.css?version=3.9.2">

<!-- Include CSS Styles from this theme -->
<link rel="stylesheet" type="text/css" href="http://10.10.10.191/bl-themes/blogx/css/style.css?version=3.9.2">

<!-- Load Plugins: Site head -->

<!-- Robots plugin -->
</head>
<body>
```

The webpage seems to be running under **Bludit CMS** and the revealed version seems to be **3.9.2**

Upon googling the found information I found an interesting **CVE** under this version of **Bludit** : [https://rastating.github.io/bludit-brute-force-mitigation-bypass/](https://rastating.github.io/bludit-brute-force-mitigation-bypass/)

The above exploit needs at least the username parameter in order to do bruteforcing and yet we dont have any.

### Directory Fuzzing

I decided to start fuzzing the webpage for any possible leftovers or interesting files
![fuzzing](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/blunder-writeup/directory-bf.png)

After some time I find 2 interesting leads :
- /admin
- /todo.txt
