---
title: "Compiled | TryHackme | Writeup"
description: "Challenge Overview"
pubDate: "Jul 16 2026"
heroImage: "/post_img.webp"
badge: "CTF"
tags: ["tryhackme", "ctf-writeup", "cybersecurity"]
---

*Original article published on [Medium - SiapaHeker](https://siapaheker.medium.com/compiled-tryhackme-writeup-49a0ab7b1efe?source=rss-cdf10c641546------2)*

**Challenge Overview**

Strings can only help you so far.

Download the task file and get started. The binary can also be found in the AttackBox inside the /root/Rooms/Compiled/ directory.

**Note:** The binary will not execute if using the AttackBox. However, you can still solve the challenge.

Answer the questions below

What is the password?

First, i downloaded the given file, then ran it in terminal

![Image](https://cdn-images-1.medium.com/max/392/1*Lxg1r_K3bE01labzNrX0tA.png)

*Run test*

This file asks for a password. Then I used the **strings **command to find several interesting strings.

![Image](https://cdn-images-1.medium.com/max/173/1*5L9sjU2zmmVozKLCSsWQrg.png)

*Strings output*

The output contained that following strings, These strings looked suspicious, so I opened the binary in **IDA** to inspect the decompiled code.

![Image](https://cdn-images-1.medium.com/max/732/1*tAo1ROSYamuFa7KOQg1NMA.png)

*Ida decompiled code*

The relevant part of the code is shown below:

```
__isoc99_scanf("DoYouEven%sCTF", s1);

if ((strcmp(s1, "__dso_handle") < 0 ||
     strcmp(s1, "__dso_handle") > 0) &&
    !strcmp(s1, "_init"))
{
    printf("Correct!");
}
```
From the decompiled code, the program compares the input stored in **s1 **with two strings, **__dso_handle **and **_init**.

```
strcmp(s1, "__dso_handle") < 0 ||
strcmp(s1, "__dso_handle") > 0
```
That first condition is equivalent to checking that s1 is **not** equal to __dso_handle.

and the second condition is only true when **s1** is exactly equal to **_init**.

```
!strcmp(s1, "_init")
```
Therefore, the only value that satisfies the condition is _init. Then the password is **DoYouEven_init**

![Image](https://cdn-images-1.medium.com/max/396/1*8tgr9wi05caLZ4-swO4EyA.png)

*Password input*

![Image](https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=49a0ab7b1efe)
