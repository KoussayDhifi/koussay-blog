---
title: Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped
description: A PortSwigger XSS lab writeup about reflected XSS into a JavaScript string with single quote and backslash escaped.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This is the 18th lab in PortSwigger's XSS lab series. It is titled [Reflected XSS into a JavaScript string with single quote and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped).

## Investigation

From the description, there is an XSS in the search functionality in JS. We need to escape a quote that encodes maybe the single or double quote using a backslash.

We have the usual website: a blog post, and this time it is actually with a search bar.

![Blog Search Page](../assets/xss/lab18/Pasted%20image.png)

Let's try the search functionality by adding a small search item named `SANSXSS`.

![Search Test](../assets/xss/lab18/Pasted%20image%20(2).png)

After looking in the page source, we can find our inserted item in the page source.

```html
<script>
   var searchTerms = 'SANSXSS';
   document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
 </script>
```

So let's see if it is possible to escape this JS string by adding a single quote.

We can see in the following image that the single quote is being escaped using a backslash, so let's try adding another backslash. Will it work?

![Escaped Single Quote](../assets/xss/lab18/Pasted%20image%20(3).png)

After testing, I detected some behaviors. This app handles the search input, and basically it is like this:

```python
QUERY = requests.get('search')
search = ""
for i in QUERY:
    if i == "'" or i == "\\":
        search += "\\"+i
    else:
        search += i

return search
```

So the JS string itself is unescapable. Let's check the `img` tag that the script adds.

```html
<img src="/resources/images/tracker.gif?searchTerms=SANSXSS">
```

The problem with this is that if we try to escape the double quotes with the payload `"SANSXSS`, we cannot because it is URI-encoded.

## Vulnerability Finding

After some research and maybe reading the writeup, I discovered something about HTML itself, or JS executed in an HTML context in a script tag specifically.

If we have a JS string like what we have here:

```html
<script>
   var searchTerms = 'SANSXSS';
   document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
 </script>
```

If we add a `</script>` in that string, HTML will see it and say, "Oh, the script tag is done." Even if it is a string, the script tag will close, and with that we have this:

```html
<script>
   var searchTerms = 'SANSXSS</script>';
   document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

With that, we can open another script tag like this:

```html
<script>
   var searchTerms = 'SANSXSS</script><script>alert()</script>';
   document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

![Injected Script Tag](../assets/xss/lab18/Pasted%20image%20(4).png)

![Solved Lab](../assets/xss/lab18/Pasted%20image%20(5).png)

With that, we have an XSS. Weird, right? This is one of the weird HTML behaviors, actually not specifically JS. That is why the angle bracket should be escaped in a JS script in an HTML context.

## Conclusion

The labs are getting harder, maybe, but this is a great new method. I do not think that this vulnerability exists in new web apps built with frameworks such as React or Angular.
