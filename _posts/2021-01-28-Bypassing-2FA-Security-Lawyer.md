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
However, bypassing this layer is possible by using **Evilginx2**
