---
title: get aHEAD Writeup PicoCTF
description: A picoCTF web challenge writeup about using a HEAD request to retrieve the flag from HTTP response headers.
author: Koussay Dhifi
categories: [CTF, WebExploitation]
tags: [WebExploitation, PicoCTF, HTTP, Recon]
pin: true
math: true
mermaid: true
---

## Introduction

This is another easy picoCTF web challenge titled [get aHEAD](https://learn.cylabacademy.org/library/132?category=1&page=2&difficulty=1). The description says the following: **Find the flag being held on this server to get ahead of the competition**.

## Thinking Process and Solving the Lab

From the name of the lab, without opening it, I would directly start with a HEAD request, which is a type of HTTP request like GET, POST, DELETE, and UPDATE. However, this request only asks for the headers of the webpage, which contain metadata about it.

So, by just using `curl` in Linux, as shown:

```sh
$ curl -I http://wily-courier.picoctf.net:56179/
HTTP/1.1 200 OK
Date: Thu, 18 Jun 2026 14:29:00 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.2.34
flag: picoCTF{Find_Your_Own}
Content-Type: text/html; charset=UTF-8
```

We get the flag directly, and thus the challenge is solved with no need to waste time.

## Conclusion

This was an easy or even peaceful challenge, if we talk in Minecraft terms, that will help beginners get to know HEAD requests in HTTP and how they can be useful during recon.
