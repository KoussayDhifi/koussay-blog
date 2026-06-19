---
title: Reflected XSS into a Template Literal with Quotes and Backticks Unicode-Escaped
description: A PortSwigger XSS lab writeup about reflected XSS inside a JavaScript template literal with key characters Unicode-escaped.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This is the [21st lab](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped) in the XSS PortSwigger labs.

## Investigation

We can see that we have the same usual website: a blog post and a search bar. The lab indicates that there is a reflected XSS in the search functionality, so let's see what is going on.

![Blog Search Page](../assets/xss/lab21/Pasted%20image.png)

Let's try to insert something like `<nice>` in the search bar.

![Search Test](../assets/xss/lab21/Pasted%20image%20(2).png)

So let's inspect the source code and see what is going on.

```html
<script>
      var message = `0 search results for '\u003cnice\u003e'`;
      document.getElementById('searchMessage').innerText = message;
</script>
```

## Vulnerability

So we have a user-controlled JS string inside a backtick. Backticks are an ES6 feature that allows developers to write JS inside a string by unpacking variables like this:

```js
let a = 3;

let msg = `This is a nice msg ${a}`

// It will unpack a and print it.
```

Inside that `${}`, we can write almost any JS syntax, so we can make this `${alert()}`.

![Template Literal Payload](../assets/xss/lab21/Pasted%20image%20(3).png)

And we solved the lab.

![Solved Lab](../assets/xss/lab21/Pasted%20image%20(4).png)

## Conclusion

This was a bug we may find in beginner JS code bases, and there are a lot of them.
