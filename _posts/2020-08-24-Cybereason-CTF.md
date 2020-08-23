---
layout: single
title: "Cybereason CTF Writeup (ALL CHALLENGES)"
excerpt: ""
date: 2020-08-24
header:
  teaser: /assets/images/xxe-injection/xxe.png
  teaser_home_page: true
classes: wide
categories:
  - ctf
tags:
  - web-app
  - stego
  - reverse-engineering
  - dfir
---

# 00 Basic
```
   ├── 00 Basic
   │   ├── Been There (5)
   │   ├── Executive retreat to Amity Island (5)
   |   └── Malop (5)
```


# 1.Been There (5)

#### Description :
``Carl, the guy a few desks down that is always talking about owls, forgot his password again and we can't reset it. But, we managed to get a lsass dump from the machine. Our password policy only requires two numbers and some letters.``
#### Files :
``lsass.dmp``

From the description and the file provided , it seems like a basic challenge. Simply downloading the dump and extracting a hash which should be decrypted afterwards.
I quickly downloaded the file and loaded it using `pypykatz`

```
┌──(root㉿kali)-[~/Downloads]
└─# ls -la lsass.DMP
-rw-r--r-- 1 root root 0 Aug 23 13:56 lsass.DMP
                                                                                                                                                               
┌──(root㉿kali)-[~/Downloads]
└─# pypykatz lsa minidump lsass.DMP
INFO:root:Parsing file lsass.DMP
FILE: ======== lsass.DMP =======
== LogonSession ==
authentication_id 1327349 (1440f5)
session_id 2
username Ben Hacked
domainname DESKTOP-QF68C1O
logon_server WIN-923HNK120UR
logon_time 2020-07-23T20:05:52.518371+00:00
sid S-1-5-21-1044796357-2890486572-795209491-1001
luid 1327349
        == MSV ==
                Username: Ben Hacked
                Domain: DESKTOP-QF68C1O
                LM: NA
                NT: 1b07bf7e4ae05f62c852daa3a8d92d25
                SHA1: 68dcb2168fe348fde9ee7beb3471658991ed812f
        == WDIGEST [1440f5]==
                username Ben Hacked
                domainname DESKTOP-QF68C1O
                password None
        == Kerberos ==
                Username: Ben Hacked
                Domain: DESKTOP-QF68C1O
                Password: None
        == WDIGEST [1440f5]==
                username Ben Hacked
                domainname DESKTOP-QF68C1O
                password None
        == DPAPI [1440f5]==
                luid 1327349
                key_guid f2af18ab-d372-4112-8fd9-dccfc34a02c
                masterkey 7a3ab1431df02bd185db2f2120716c439f4a76abd2ac4c55d00045ab83c74bde34f1379a756f20b5190cf8487fa681b674622164149a6009982923308618dfc8
                sha1_masterkey bf5a5d35844dd6ac7a925ba62d548ec34121dd44
```

I can simply notice the NT hash : `1b07bf7e4ae05f62c852daa3a8d92d25`
However while trying to crack on john or hashcat I didnt came up with anything so I decided to use https://hashes.com to crack it and came up with the following result:
![hash](url)

#### Flag : `archimedes01`
