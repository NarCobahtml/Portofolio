---
title: "TakeOver | TryHackMe | Writeup"
description: "This challenge revolves around subdomain enumeration."
pubDate: "Jul 14 2026"
heroImage: "/post_img.webp"
badge: "CTF"
tags: ["tryhackme", "ctf-writeup", "ctf", "tryhackme-walkthrough", "cybersecurity"]
---

*Original article published on [Medium - SiapaHeker](https://siapaheker.medium.com/takeover-tryhackme-writeup-b547326910e0?source=rss-cdf10c641546------2)*

### **Challenge Overview**

This challenge revolves around subdomain enumeration.

Hello there,

I am the CEO and one of the co-founders of futurevera.thm. In Futurevera, we believe that the future is in space. We do a lot of space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.

Recently blackhat hackers approached us saying they could takeover and are asking us for a big ransom. Please help us to find what they can takeover.

Our website is located at [https://futurevera.thm(opens in new tab)](https://futurevera.thm/)

Hint: Don’t forget to add the 10.48.152.105 in /etc/hosts for futurevera.thm ; )

**Recon**

The challenge description mentions that the target website is available at [**https://futurevera.thm**](https://futurevera.thm) and reminds us to add the provided IP address to our /etc/hosts file before accessing the website.

After configuring the hosts file, I started by performing an **Nmap **scan to identify the exposed services.

```
nmap <YOUR TARGET IP>
```
![Image](https://cdn-images-1.medium.com/max/637/1*edV4qU9meex-cFruPOnmtw.png)

*Nmap output*

The scan revealed three open ports: SSH (22), HTTP (80), and HTTPS (443). Since both web services are available and the challenge description hints at web enumeration, the next step is to investigate the website further.

**Enumeration**

According to the challenge title and description, the objective revolves around discovering hidden subdomains. Therefore, I started by performing virtual host enumeration using **ffuf**.

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt \
-u https://<YOUR TARGET IP>/ \
-H "Host: FUZZ.futurevera.thm" -mc 200,300,3001
```
![Image](https://cdn-images-1.medium.com/max/865/1*x7fQE8_-NqfgYFXg0LFxpQ.png)

*ffuf raw*

The result showed almost every entry in the wordlist returned a **200 OK** response, this result looked suspicious because it is highly unlikely that hundreds of subdomains actually exist.

To verify this behavior, I compared the response of one of the discovered subdomains with a completely random, non existent subdomain using **curl**.

![Image](https://cdn-images-1.medium.com/max/792/1*--BjI327xhc7SyuDCBMKIQ.png)

*curl response size 4605*

Both requests returned exactly the same response size 4605

To eliminate these false positives, I filtered out responses with a size of **4605 bytes** using the -fs (filter size) option.

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt \
-u https://<YOUR TARGET IP>/ \
-H "Host: FUZZ.futurevera.thm" -mc 200,300,3001 \
-fs 4605
```
![Image](https://cdn-images-1.medium.com/max/873/1*Le-8ZdiuKmxi7Hh83qAG0Q.png)

*ffuf filter output*

*After discovering the ****blog**** and ****support**** subdomains, I began manually inspecting both websites. I reviewed the page source, checked the available functionality, and monitored the network traffic using the browser’s developer tools. However, neither the source code nor the HTTP requests revealed anything useful on the ****blog**** subdomain.*

*I then turned my attention to the ****support**** subdomain. Similar to the blog, there was nothing immediately suspicious in the page content or network traffic. As a next step, I examined the website’s SSL certificate and noticed an additional hostname listed in the ****Subject Alternative Name (SAN)**** field.*

![Image](https://cdn-images-1.medium.com/max/650/1*yiLMhFrKYJRnGilSqAbGPQ.png)

*ssl certificate*

I added this newly discovered hostname to my /etc/hosts file and visited it in the browser. The website looked normal, and after I inspected the website by reviewing the page source, exploring the available functionality, and analyzing the HTTP requests using the browser’s developer tools. However, no interesting information or hidden functionality was identified.

To investigate further, I also tested the host using curl. Interestingly, requesting the host over **HTTPS** did not reveal anything unusual.

![Image](https://cdn-images-1.medium.com/max/1024/1*EgNNzRRYBQ5WtpQOishabw.png)

*curl https output*

However, when I sent the request **without specifying HTTPS**, the server responded differently and exposed the challenge flag.

```
curl -i secrethelpdesk934752.support.futurevera.thm
```
**Flag Captured**

![Image](https://cdn-images-1.medium.com/max/1024/1*YTXFa0oYy2dF_BpuGppPkA.png)

*curl output flag*

![Image](https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=b547326910e0)
