---
layout: single
title: "Vulnhub Write-up #1 : Stripes"
excerpt: "Stripe is a easy to medium linux machine with a little OSINT in the beggining and escalating to user by first escaping a shell. The second user is allowed to run a sudo command which gives us root access..."
date: 2020-04-24
header:
  teaser: /assets/images/Stripe-Vulnhub/vulnhub.png
  teaser_home_page: true
classes: wide
categories:
  - vulnhub
  - infosec
tags:
  - busctl  
  - wordpress
  - restricted-shell
---



I made a little break from HTB to try out a boot2root machine that a friend of me recently released on Vulnhub. Stripe is a easy to medium linux machine with a little OSINT in the beggining and escalating to user by first escaping a shell. The second user is allowed to run a sudo command which gives us root access...

## Summary 

- Enumerate the webpage to grab the credentials
- Escape the jail shell
- Escalate to the second user through reused password
- Abuse a bash script that we can run as root 

## Enumeration 
### Nmap :

![nmap](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/1.png)


From the initial nmap scan I could notice only 2 ports open : 22 (ssh) , 80 (http) . 
In this case the web page should be the initial recon for this machine.

I visited the webpage and it seemed a little ugly and the redirections didnt work : 

![site](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/2.png)

While trying to click some buttons I got redirected to http://stripes/ . In this case we have to add it to our hosts and fully navigate the site.
After adding it to /etc/hosts under the local ip the website started to look complete : 

![fixed](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/3.png)

So far I could notice 2 blog posts from different users. While enumerating them I noticed the post from wp-admin was a little usseles so I focused more on the post from the other user (Joe Maldonado) :

![post](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/4.png)

There is some clues about his preferences , favourite words and his birthday too which makes it look like an OSINT game.
Being it a wordpress site , my first guess was to grab the marked words and create some wordlists or guess the password for joe , but first I would have to grab both usernames to check for a valid login.

I could simply do that by navigating into each user's profile and check the url path :

![users](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/5.png)
![user2](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/6.png)

- wp-user 
- joem

I verified both usernames through admin login page and both returned to be valid :

![verify](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/7.png)

So far, I tried to manually guess the password from the clues I had and try against both users but no results. 

After a lot of attempts I did by trying to login to the wordpress site , I went to try against ssh by simply using one of joe favourite words + birthday as a password...

Both combinations didnt seem to work : 
- ```1. joem:tigris1963```
- ```2. joem:exotic1963```

However I decided to only use ```joe``` as username and ... We have results : 

![ssh](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/8.png)

```joe:tigris1963``` worked fine with the ssh client !

After some little enumeration I did , I could notice there was another user named carole , but before that I noticed we are in a jail. Joe's shell is very restricted and I could run just few commands which could help me escape :

![shell](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/9.png)

But I see 2 very good clues here :

- export
- busctl

I can probably export the shell to /bin/sh and abuse busctl to launch an interactive shell : 

![escape](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/10.png)

So as seen above I was able to export the shell and drop one through busctl as it uses less pager to display info and we can abuse it to spawn a shell.
After getting a full shell as joe my first guess was checking at the wordpress site config and the database to gather some information for the other user .

## User Escalation 

From some researching in google I found out that wordpress files are usually stored undes /srv folder and I was able to dig in. 

![config](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/11.png)

By looking around config I can see the database password and the user which seems to be wp-user .
I tried to dump the mysql database but returned with nothing interesting so I returned to the first blog from wp-user which seems to be Carole :

![carole](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/12.png)

So I can maybe login in machine using the same database password ? :

![su](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/13.png)

Yeah I could , and from here I can grab the user flag :

```stripes{c0n6r475_bu7_7h15_15n7_r34l_fl46}```

## Root Escalation (Unintended)

Simple enumerating returned with interesting information :

  Under Carole folder there seems to be a bash script sending a message to ```joe``` !
  
  Using ```sudo -l``` returns that I can also run it as sudo :

![sudo](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/14.png)

We can edit the script so my guess was to simply edit the bash script to a reverse shell and run as sudo which could return me a root shell in my listener : 

![script](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/15.png)

I opened a netcat listener in another shell and simply ran the script using sudo because I can do that : 

![listener](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/16.png)

I was able to simply get a root shell and grab the prize (flag) .

While retrying several ways to I also found one more unintended root before finally figuring out the intended root.

![scr](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/17.png)

While reading the script I could simply play with the message path by adding a symlink to ``/root/root.txt`` :

![symlink](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/18.png)

and.. while it executes it reads the root flag as a sending message since I made a symlink of the flag with the message...

![flag](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/19.png)


## Intended Root 

After sometime of analyzing the code I saw that I can also abuse the eval function and call system commands through it, which was also meant to be the intended root privesc for this machine ...
I can edit the ``msg4joe`` file and add commands to it :

 - ```;'id' >> /tmp/id```
 
 We simply inject command ``id`` to be executed and send results to ``/tmp/id`` : 
 
 ![eval](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/Stripe-Vulnhub/20.png)



This was my first time trying a Vulnhub machine and this box learned me some new tricks of escaping shells.
Thanks to Kyn, I will now take a closer look to vulnhub machines which seem very interesting too...and as always 
THANK YOU for reading :)
