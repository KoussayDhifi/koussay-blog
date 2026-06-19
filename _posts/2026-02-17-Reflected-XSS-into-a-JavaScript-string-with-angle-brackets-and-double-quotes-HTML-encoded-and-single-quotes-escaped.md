---
title: Reflected XSS into a JavaScript String with Angle Brackets and Double Quotes HTML-Encoded and Single Quotes Escaped
description: A PortSwigger XSS lab writeup about escaping a JavaScript string when angle brackets and double quotes are HTML-encoded and single quotes are escaped.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This lab is titled [Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped). If you read or tried the previous lab, [18th](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped), we had a vulnerability where the angle brackets were not escaped in JS strings in an HTML context. But this time, they are escaped, so what will we do here?

## Investigation

We have the same blog with a search bar as usual.

![Blog Search Page](../assets/xss/lab19/Pasted%20image.png)

We have basically the same thing as the previous app, so when we search for an article and check the source code, we can see this:

```html
 <script>
 var searchTerms = '&lt;/script&gt;';
 document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

As you can see here, I tried to add `</script>`, and it is escaped or encoded. Everything is encoded. You may be asking how we are going to solve this. Get ready, because I do not know either.

## Vulnerability and Payload

When we read the description of the challenge, we realize that the backslash is not encoded, and therefore we can escape the JS string by using the payload `sans';alert();//`.

![Payload in Search](../assets/xss/lab19/Pasted%20image%20(2).png)

```html
<script>
                        var searchTerms = 'sans\\'; alert(); //';
                        document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
                    </script>
```

With that, we get the alert.

![Solved Lab](../assets/xss/lab19/Pasted%20image%20(3).png)

## Conclusion

That was a pretty straightforward lab from the series of labs 18-19. We learned different methods for escaping a JS string, from injecting angle brackets into JS in an HTML context to escaping strings using a backslash.
