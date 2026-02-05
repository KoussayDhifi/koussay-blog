---
title: From Swagger to Secrets: Exploiting an Exposed Heap Dump
description: A writeup for the solution of the head-dump challenge in PicoCTF.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, PicoCTF, CTFs, HEAD]
pin: true
math: true
mermaid: true
---

## Introduction

This is a writeup for the easy web-exploitation challenge in picoCTF named [head-dump](https://play.picoctf.org/practice/challenge/476?category=1&difficulty=1&page=1)


## Investigation (Recon)

We have a simple blog post like website that has various articles as shown in the following image.

![Blog Post](../assets/img/head-dump/Pasted%20image.png)

One article is siting that there is an API documentation as shown in the following image.

![Post](../assets/img/head-dump/Pasted%20image%20(2).png)

So let's try and check the documentation by clicking the link.

We can see that we are provided with the swagger of the API.

![API documentation](../assets/img/head-dump/Pasted%20image%20(3).png)

We can see we have various endpoints the usual ones to navigate the website and a section of endpoints that manages the posts with the usual CRUD operations.

But in the last section titled `Diagnosing` we have an endpoint named `/heapdump`, if we are going to think in a CTFy way the flag is lurking there since as it says in the description and the title of the project so let's check it.

![Diagnosing endpoint](../assets/img/head-dump/Pasted%20image%20(4).png)

After we 'execute' (Consume) the endpoint we can see that it returns in the body of the response a file which is a heapsnapshot.

## Heapsnapshot

A dump (snapshot) of the memory heap of a running program at a specific moment.

Basically a photo of everything currently stored in memory.

The leaking of heapsnapshot can be absolutely dangerous to any web app because you can find cruicial information like database usernames + plaintext passwords, JWT signing secrets, OAuth client secrets, API keys, Internal service tokens and more ... Because the heapsnapshot contains everything the app is storing in memory and the app will store cruicial info like those.

Usually this file is used by developers in Nodejs apps, Java(JVM heap dump) or .NET (memory dump) for debugging purposes and in nodejs for example usually it is stored in an endpoint like this.

```js

app.get('/debug/heap', (req, res) => {
  const file = v8.writeHeapSnapshot();
  res.download(file);
});


```

so in fuzzing directories always try to check for endpoints like heap or /admin/debug you may find something with enough recon.


## Solving CTF

So after downloading heapsnapshot we can see that the file contains the flag as shown with other things.

![Flag](../assets/img/head-dump/Pasted%20image%20(5).png)


You can submit it and then the lab is solved inshallah.

## Conclusion

Heapsnapshot is a new debugging style that I learned in this easy CTF challenge. Wonder how hard challenges are because all of this seems straight forward.