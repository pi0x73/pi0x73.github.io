---
layout: single
title: "Hack The Box Write-up #2 : ServMon"
excerpt: "My walkthrough of ServMon Machine from HackTheBox"
date: 2020-06-20
header:
  teaser: /assets/images/servmon-walkthrough/servmon.png
  teaser_home_page: true
classes: wide
categories:
  - hackthebox
  - infosec
tags:  
  - windows
  - cve
---

![Card](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/servmon-walkthrough/servmon.PNG)

## Enumeration

As always I started by firing up a nmap scan against the host and came up with the following results : 

```
root@kali:~# nmap -sC -sV -v -p- 10.10.10.184
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_01-18-20  12:05PM       <DIR>          Users
| ftp-syst:
|_  SYST: Windows_NT
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 b9:89:04:ae:b6:26:07:3f:61:89:75:cf:10:29:28:83 (RSA)
|   256 71:4e:6c:c0:d3:6e:57:4f:06:b8:95:3d:c7:75:57:53 (ECDSA)
|_  256 15:38:bd:75:06:71:67:7a:01:17:9c:5c:ed:4c:de:0e (ED25519)
80/tcp    open  http
| fingerprint-strings:
|   GetRequest, HTTPOptions, RTSPRequest:
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo:
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL:
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
|_http-favicon: Unknown favicon MD5: 3AEF8B29C4866F96A539730FAB53A88F
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5666/tcp  open  tcpwrapped
6063/tcp  open  x11?
6699/tcp  open  napster?
7680/tcp  open  pando-pub?
8443/tcp  open  ssl/https-alt
| fingerprint-strings:
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest:
|     HTTP/1.1 302
|     Content-Length: 0
|     Location: /index.html
|     refo
|_    2-contai
| http-methods:
|_  Supported Methods: GET
| http-title: NSClient++
|_Requested resource was /index.html
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-01-14T13:24:20
| Not valid after:  2021-01-13T13:24:20
| MD5:   1d03 0c40 5b7a 0f6d d8c8 78e3 cba7 38b4
|_SHA-1: 7083 bd82 b4b0 f9c0 cc9c 5019 2f9f 9291 4694 8334
|_ssl-date: TLS randomness does not represent time
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
```

There seem to be a few holes that could get me inside the machine , but the most suspicious of them where I could gather a little more information for free , seems that allowed `anonymous login` on the ftp server.

After logging in by making use of the anonymous ftp login I was presented with the users folder (Nathan and Nadine) , with both of them having a note saved in text files in their folders : ``Confidental.txt`` and ``Notes to do.txt`` with the following content :

- Confidental.txt
```
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine
```

- Notes to do.txt
```
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```

There seems to be an useful information for some passwords stored on Nathan Desktop folder which we should keep in mind.

### Web

After gathering enough initial information it's safe to take a look on the webpage which represents a simple login page :

![web](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/servmon-walkthrough/web-servmon.png)

The webpage seems like an administration panel for a CCTV system so for the sake of simplicity I decided to search for ``NVMS-1000`` on msf to see if there was any ready made exploit that I could make use of. (not useful for OSCP tho) 

```console
msf5 > search nvms

Matching Modules
================

   #  Name                                       Disclosure Date  Rank    Check  Description
   -  ----                                       ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/tvt_nvms_traversal  2019-12-12       normal  No     TVT NVMS-1000 Directory Traversal
```
   
The dots starting to connect!
I came up with a directory traversal exploit for the following NVMS version that could help me grab the previously mentioned ``Passwords.txt`` from Nathan's desktop folder.

