---
title: Stored XSS into HTML context with nothing encoded
description: This writeup delves into the second XSS lab in port swigger.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, Labs]
pin: true
math: true
mermaid: true
---

## Introduction
So, a new lab as part of my XSS diaries... Don't expect anything professional from this—just me, a lab, and trying to break it.
This is the second lab of XSS in portswigger named [Stored XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded).

## Investigation and Fuzzing
Another blog, but this time it has no search bar and has almost the same structure as the other one. So I think injecting payloads in comments will come in handy this time.

![Blog Image](../assets/img/xssLabs/Lab2/ImageOne.png)


Let's inject the famous payload in the comment, but let's start with the simplest one: `<script>alert(0)</script>`. 

![Blog Image](../assets/img/xssLabs/Lab2/ImageTwo.png)

And that's it!!! We found a stored XSS vulnerability for the comment input. 

![Blog Image](../assets/img/xssLabs/Lab2/ImageThree.png)


Let's try other inputs like name or email. Hmmmm, the name field is sanitized as shown here in Burp Suite: `&lt;script&gt;alert(0)&lt;/script&gt;`

## Conclusion

That was a straight-forward beginner friendly lab. The next labs will have more challenges I think.