---
title: Reflected XSS in a Canonical Link Tag
description: A PortSwigger XSS lab writeup about reflected XSS in a canonical link tag.
author: Koussay Dhifi
categories: [Vulnerabilities, WebExploitation]
tags: [WebExploitation, XSS, PortSwigger, Labs]
pin: true
math: true
mermaid: true
---

## Introduction

This is the [17th lab](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag) in this PortSwigger XSS series.

The description of the lab indicates that it reflects user input in a canonical link tag and escapes angle brackets.

## Canonical Link

It is an attribute in the head of the HTML to indicate the canonical URL of a certain page to index if that page repeats.

```html
<link rel="canonical" href="www.site.com/shoes">
```

Sometimes we need it when we have URLs like these:

```text
www.site.com/shoes?q=red
www.site.com/shoes?q=blue
```

So if those URLs get indexed one by one, without indicating the canonical URL of those pages, we will have a problem with SEO.

## Investigation

We have the usual blog that PortSwigger provides us with.

![Blog Page](../assets/xss/lab17/Pasted%20image.png)

Since the canonical link is used in pages that contain variants of the URL, we can guess that the user-controlled canonical link is in the view post pages because they have the following URL structure: `https://portswigger/post?postId=5`.

So when we check the source code of a specific post page, we can find it.

![Canonical Link Source](../assets/xss/lab17/Pasted%20image%20(2).png)

We have it here:

```html
<link rel="canonical" href='https://0a08000a0494f0d08080949d006e00ae.web-security-academy.net/post?postId=5'/>
```

## Vulnerability Detection

The canonical name is user-controlled, so by just editing the URL, we can escape the link tag.

When I try to inject something in `postId` like this:

```text
https://0a08000a0494f0d08080949d006e00ae.web-security-academy.net/post?postId=5' onload='alert(1)
```

I get an error that tells me `postId` is invalid.

![Invalid Post ID](../assets/xss/lab17/Pasted%20image%20(3).png)

So I need to also escape `postId` by adding another argument. Let's call it `ora`.

```text
https://0a08000a0494f0d08080949d006e00ae.web-security-academy.net/post?postId=5&ora=5'onload='alert(1)
```

With that, the page loads successfully, and we can see that we indeed escaped the `href` in the link tag.

![Escaped Canonical href](../assets/xss/lab17/Pasted%20image%20(4).png)

But the `onload` did not work. We need to add another attribute that fires JS. I tried to add a `<script>` tag, but the less-than and greater-than signs are encoded. The lab indicates that we need to add an attribute to the link to fire JS, and we have this kind of hint:

> To assist with your exploit, you can assume that the simulated user will press the following key combinations: ALT+SHIFT+X, CTRL+ALT+X, Alt+X.

## Searching

I did not understand how those combinations would help us. They did nothing. I tried them and asked ChatGPT and Google, but no one had an answer.

After reading the writeup, I found a solution, which is a hypothetical one.

## Exploitation and Payload Crafting

In HTML, there is an attribute that you add to HTML elements named `accesskey`, like this:

```html
<a accesskey="x">Link</a>
```

So when you fire that access key, that element will be focused on, and you fire that element depending on the operating system and the browser you have. For Chrome, you use the following combinations:

- `ALT+SHIFT+X`: Windows
- `CTRL+ALT+X`: macOS
- `Alt+X`: Linux

This `accesskey` attribute is addable to the link tag, although it is not rendered.

```html
<link accesskey="x" onclick="alert(1)"/>
```

With that, the URL is:

```text
https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

By pressing `Alt+X`, because I use Linux, I get the alert, and the lab is solved.

## Conclusion

Is this case likely to happen? No, but it is a good theoretical exercise to indicate that XSS is possible even with non-rendered elements.
