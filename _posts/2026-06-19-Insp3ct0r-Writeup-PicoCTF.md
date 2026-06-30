---
title: Insp3ct0r Writeup PicoCTF
description: A writeup for the PicoCTF easy web challenge named Insp3ct0r.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, CTFs, Recon, Enumeration]
mermaid: true
---

## Introduction

This is another easy web PicoCTF challenge titled [Insp3ct0r](https://learn.cylabacademy.org/library/18?category=1&page=2&difficulty=1). It has the following description: **Kishor Balan tipped us off that the following code may need inspection.**

Like all easy challenges, this one focuses primarily on recon.

## Recon

When we open the challenge, we get a website that looks very similar to one of the previous challenges named [Scavenger Hunt](https://learn.cylabacademy.org/library/161?category=1&page=2&difficulty=1). It has a section detailing "what" and "how", as shown in the following images.

![Website Overview](../assets/insp3ct0r/Pasted%20image.png)

![What and How Sections](../assets/insp3ct0r/Pasted%20image%20(2).png)

When we check the source code of the page, we find the first part of the flag:

```html
        <!-- Html is neat. Anyways have 1/3 of the flag: picoCTF{tru3_d3 -->
```

So intuitively, we should check the stylesheets and JS files, as they might contain parts 2 and 3.

And indeed, the stylesheet `mycss.css` contains the following comment:

```css
/* You need CSS to make pretty pages. Here's part 2/3 of the flag: t3ct1ve_0r_ju5t */
```

And when we check the JS file, we find the last part of the flag:

```js
/* Javascript sure is neat. Anyways part 3/3 of the flag: _lucky?FindYourOwn} */
```

With that, the lab is solved.

## Conclusion

This challenge is a much easier version of [Scavenger Hunt](https://learn.cylabacademy.org/library/161?category=1&page=2&difficulty=1). If you want a slightly more challenging version, definitely check that one out.
