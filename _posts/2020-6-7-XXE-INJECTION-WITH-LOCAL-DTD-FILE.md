---
layout: single
title: "XXE injection with local DTD file and PHP filter."
excerpt: "A short brief explaination + walkthrough for XXE injection attack and how it works."
date: 2020-06-07
header:
  teaser: /assets/images/Stripe-Vulnhub/vulnhub.png
  teaser_home_page: true
classes: wide
categories:
  - bug-hunting
tags:
  - web-app
  - xxe
  - file-read
---

In this short article I'm going to make a short explaination of the XXE injection, how it works and how it can be used into any vulnerable app using a local DTD file.

First lets take a look from an usual dummy paragraph of what a XXE injection is...

## What is a XML external entity (XXE) injection? 
XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any backend or external systems that the application itself can access.

In some situations, an attacker can escalate an XXE attack to compromise the underlying server or other backend infrastructure, by leveraging the XXE vulnerability to perform server-side request forgery (SSRF) attacks.

## What are some types of a XXE attack?

There are various types of XXE attacks:

- Exploiting XXE to retrieve files, where an external entity is defined containing the contents of a file, and returned in the  
  application's response.
- Exploiting XXE to perform SSRF attacks, where an external entity is defined based on a URL to a back-end system.
- Exploiting blind XXE exfiltrate data out-of-band, where sensitive data is transmitted from the application server to a system that the   attacker controls.
- Exploiting blind XXE to retrieve data via error messages, where the attacker can trigger a parsing error message containing sensitive   data.

In this article I'm going to explain how we can retrieve files from the system through a vulnerable doc-upload web application.
Lately I came through an interesting machine in HackTheBox (Patents) which represented a simple website with an upload form where I could upload docx files to the web server.

My first thought when I saw I could upload ``.docx`` files was that I could possibly inject macro code or usual injections to have a possible ``Remote Command Execution`` to the server but as expected it wasn't meant to be that easy, so I started to dig in more in Google to find new possibilities of injection attacks through .docx files.

While searching a came accross https://portswigger.net/web-security/xxe which helped me the most to understand the basics of a XXE attack and later I could use https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection to test some payloads in the remote server and see if it could work.

## Exploitation 
First I went to https://docs.google.com/ to create a .docx and download it to my machine to make further edits to it and inject some XML code.

![docs](image)

After it is downloaded I can unzip the document and make edits to the xml file:

![unzip](image)

We see a bunch of xml files that together make the `docx` file functionable. 
From here we can create a folder named `CustomXML` and put our malicious xml files inside ``item1.xml`` , ``item2.xml`` and so on.

So first I created the folder and used a payload from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#xxe-oob-with-dtd-and-php-filter) and injected it into the ``item1.xml``
