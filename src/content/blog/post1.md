---
title: "PIE TIME | CyLab | Writeup"
description: "Can you try to get the flag? Beware we have PIE! Connect to the program with netcat:"
pubDate: "Nov 05 2026"
heroImage: "/post_img.webp"
badge: "CTF"
tags: ["ctf-writeup", "ctf", "ctf-walkthrough", "binary-exploitation", "cybersecurity"]
---

*Original article published on [Medium - SiapaHeker](https://siapaheker.medium.com/pie-time-cylab-writeup-b43d39550d06?source=rss-cdf10c641546------2)*

### Challenge Overview

Can you try to get the flag? Beware we have PIE! Connect to the program with netcat:

```
nc xebec.cylabacademy.net <PORT>
```
The program’s source code and the binary are provided.

Hints:

- Can you figure out what changed between the address you found locally and in the server output?

![Image](https://cdn-images-1.medium.com/max/662/1*b_cyq4ySR2mslGlT5i6P1g.png)

*challenge overview*

First, i downloaded the given binary and checked the file type using the **file **command.

![Image](https://cdn-images-1.medium.com/max/1024/1*xpWXq6_AqjsRfUFsnqj6GA.png)

*file output*

The binary is a 64-bit PIE executable and it is not stripped, so the function names are still visible. Next, i checked the protections using **checksec**.

```
checksec --file=vuln
```
![Image](https://cdn-images-1.medium.com/max/1024/1*ytxiMztsmDp85MfyNlOH7Q.png)

*checksec output*

The output shows Full RELRO, stack canary, NX, and PIE are all enabled. Since PIE is enabled, the base address of the binary is randomized every time it runs, so we cannot simply hardcode a function address.

Then i ran the binary locally.

![Image](https://cdn-images-1.medium.com/max/523/1*b3xZY6R4qg1PmUPG4s9vrA.png)

*run test*

The program prints the address of main, then asks for an address to jump to. When i entered a random address (0x12345), it crashed with a segfault. This tells us the program really jumps to whatever address we give it.

To understand it better, i opened the binary in pwndbg and listed the functions.

```
pwndbg> info function
```
![Image](https://cdn-images-1.medium.com/max/393/1*wMux7GRgO7zlTsEO7QOcVQ.png)

*info function output*

There is an interesting function called **win**. Here are the offsets that matter: **win 0x12aa** and **main 0x134c**

Then i disassembled main.

![Image](https://cdn-images-1.medium.com/max/1014/1*nMsr3rPiUMmXrZ2yLn6qtQ.png)

The relevant part of the code:

- **prinf **prints the address of main (this is our leak)
- **scanf** reads our input into a variable
- **call rax** executes the address we entered

So there is no buffer overflow here. We only need to provide the correct address of **win**. A quick look at **win **shows it opens a file and prints its content, which is exactly what we want.

![Image](https://cdn-images-1.medium.com/max/735/1*Hw3dncNB-UysP4p6AspTYg.png)

*disassemble win output*

PIE only randomizes the base address, the distance between functions inside the binary stays the same. Because the program leaks the address of main, we can calculate **win **from it. First, calculate the difference between the two functions:

```
p/x &main - &win
```
![Image](https://cdn-images-1.medium.com/max/270/1*OttaGPgffpat1brXpuwqZw.png)

*offset calculation*

Now connect to the server and get the leaked address.

![Image](https://cdn-images-1.medium.com/max/435/1*0kV2d0qh8J9Ke9tUNvsTJw.png)

Then subtract the offset from the leaked address:

```
p/x 0x62306d04834c - 0xa2
```
![Image](https://cdn-images-1.medium.com/max/352/1*8fziOpOS8wErr60aUexSPg.png)

Finally, send the calculated address as the input. Note that the address changes on every new connection, so the calculation must be done in the same session.

![Image](https://cdn-images-1.medium.com/max/587/1*K6IbgkBjo5ZrsoZdBJUZsA.png)

*flag output*

and there we goo, we got the flag!!

```
academy{REDACTED}
```
![Image](https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=b43d39550d06)
