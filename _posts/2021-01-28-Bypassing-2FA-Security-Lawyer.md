---
layout: single
title: "Bypassing 2FA (two-factor authentication) through a MITM attack"
excerpt: "A walkthrough demonstrating how secure 2FA is and how can it be bypassed."
date: 2021-01-28
toc: true
toc_sticky: true
header:
  tagline: "A walkthrough demonstrating how secure 2FA is and how can it be bypassed."
  overlay_image: /assets/images/background.jpg
  overlay_filter: 0.5
  teaser: /assets/images/buff-writeup/buff.png
  teaser_home_page: true
  
tags:  
  - phishing
  - 2fa
  - reverse-proxy
  - mitm-attack
---
Phishing is yet the most used and popular technique day-to-day which has brought compromises from single user accounts to large companies.  
While looking for suggestions , the most common recommendation given nowadays is to use the **two-factor-authentication** layer to defend from various phishing methods.   
But how secure is **2FA** itself?  
In this blog post we'll look on how **2FA** can be bypassed in an usual phishing situation and how to set-up [Evilginx2](https://github.com/kgretzky/evilginx2)

## Required Stuff
- a custom domain that you control
- a VPS (Virtual Private Server)
- Evilginx2

## What is 2FA? 
Two-factor authentication or 2FA, is an electronic authentication method in which a computer user is granted access to a website or application only after successfully presenting two or more pieces of evidence (or factors) to an authentication mechanism: knowledge (something only the user knows), possession (something only the user has), and inherence (something only the user is).


By reading this short explaination about **two-factor-authentication** you would assume that the mechanisms used look secure and would make it almost impossible for a hacker to break through.  

However, bypassing this layer is possible by using **Evilginx2**, a Man in the Middle attack framework which is able to act as a reverse proxy and help the attacker get through verification steps.

![background](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx1.jpg)

## Installation 

>*Note : in this part I am not going to cover setting up the VPS and domain but simply setting up the tool in a linux environiment and using it over a quick and short demo.*

Firstly we would want to grab a pre-compiled binary from the official repo : 

```sh
pi0x73@evilginx2:~$ wget https://github.com/kgretzky/evilginx2/releases/download/2.4.0/evilginx-linux-amd64.tar.gz
--2021-01-28 18:58:03--  https://github.com/kgretzky/evilginx2/releases/download/2.4.0/evilginx-linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.121.3
Connecting to github.com (github.com)|140.82.121.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
[...]
evilginx-linux-amd64.tar.gz               100%[====================================================================================>]   8.24M  3.24MB/s    in 2.5s    

2021-01-28 18:58:06 (3.24 MB/s) - ‘evilginx-linux-amd64.tar.gz’ saved [8636795/8636795]

pi0x73@evilginx2:~$ 
```

Once the package has been download we can proceed by unpacking and running the installation script. **golang** is a required package as well so I'm going to install that too :

```sh
pi0x73@evilginx2:~$ sudo apt install golang
Reading package lists... Done
Building dependency tree       
Reading state information... Done
golang is already the newest version (2:1.10~4ubuntu1).
0 upgraded, 0 newly installed, 0 to remove and 9 not upgraded.
pi0x73@evilginx2:~$ tar -xf evilginx-linux-amd64.tar.gz
pi0x73@evilginx2:~$ cd evilginx/
pi0x73@evilginx2:~/evilginx$ ls
evilginx  install.sh  phishlets  templates
pi0x73@evilginx2:~/evilginx$ sudo bash install.sh
```
After that we can simply fire it up by running ``sudo evilginx2`` anywhere on the terminal :

![run](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx2.png)

Before firing up a phishing page **Evilginx** requires some additional setup such as setting our custom domain , our public ip etc... 

![config](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx43.png)

These 3 entries appearing in the screen have to be added in the DNS config for our domain as shown in the following screenshot :

![DNS](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx4.png)

The configuration should take somewhere to 15 minutes to apply in the domain. You can confirm the changes by running ``nslookup yourdomain`` in another terminal :

```sh
┌──(suljot㉿kali)-[~]
└─$ nslookup pi0x73.cf
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   pi0x73.cf
Address: 20.77.25.9
```

When it's all set-up and looking good , you can fire a phishing page to be generated in your domain. (I used twitter)
![phishlet](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx5.png)

This **URL** is supposed to be sent to the target which in this case is myself.   

![template](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx6.png)

The phishing page looks just like the real one so I am going to treat it as an user who is right about to get compromised. I entered my credentials on it and clicked **Login**

![2fac](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/2FA-Bypass/evilginx7.png)

