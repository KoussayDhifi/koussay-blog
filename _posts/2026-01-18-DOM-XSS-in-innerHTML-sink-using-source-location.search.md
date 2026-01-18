---
title: DOM XSS in innerHTML sink using source location.search
description: This writeup delves into the fourth XSS lab in port swigger.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

So this is another stand user in my journey in XSSLand. Let's see how are we going to beat it -INSHALLAH-
The lab title is [DOM XSS in innerHTML sink using source location.search](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink)

## Investigation

This time our friendly search bar has came back and we have almost the same structure as the previous ones.

![Blog Image](../assets/img/xssLabs/Lab4/ImageOne.png)

### Search Bar

Let's try and see if the search bar works as always or not.

So it seems the same but does the search query is sent to the server? or the browser?? `?search=hey` Hmm let's check the source code and intercept the traffic with burpsuite. So after checking the code source of the web page we find this script
```js
  <script>
        function doSearchQuery(query) {
            document.getElementById('searchMessage').innerHTML = query;
        }
        var query = (new URLSearchParams(window.location.search)).get('search');
        if(query) {
            doSearchQuery(query);
        }
    </script>

```

So we realize that the search param is not a request to the server but rather the browser takes the param of search and queries it, so basically the input=query here and it is inserted to the DOM with innerHTML method. While this method give some sense of security since it does not allow the execution of inserted `<script>` tag. But it can be leveraged from using the famous payload : `<img src=x onerror=alert(1)>` since innerHTML executes DOM events. And with that we get our DOM-XSS

![Blog Image](../assets/img/xssLabs/Lab4/ImageTwo.png)


## Conclusion

What a small and easy stand user...can't wait for the other one.
