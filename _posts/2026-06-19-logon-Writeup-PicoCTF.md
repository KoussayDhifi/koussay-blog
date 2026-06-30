---
title: logon Writeup PicoCTF
description: A writeup for the picoCTF easy web challenge named logon.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, CTFs, Cookies, Recon]
mermaid: true
---

## Introduction

This is another easy web picoCTF challenge titled [logon](https://learn.cylabacademy.org/library/46?category=1&page=1&solved=true). It has the following description: **The factory is hiding things from all of its users.** Again, since this is an easy lab, maybe it is recon-based rather than exploitation-based, so let's check it out.

## Recon

When we open the challenge, we find a login form asking for a username and password, alongside a navbar that has a home and sign out button, as shown in the following image.

![Login Page](../assets/logon/Pasted%20image.png)

When we write any kind of password, we get redirected to `/flag`, and it says that we are logged in but there is no flag for us, as shown in the following image.

![Logged In Without Flag](../assets/logon/Pasted%20image%20(2).png)

So either there are some hidden creds in the website, and we are going to use Gobuster to list different directories and files since it may actually be hiding something. See what I did there?

We also follow requests and response headers using Burp Suite to see if there is actually something hidden there. See what I also did there?

### Burp Investigation

When we do a simple login and intercept the traffic, we get our usual POST request, as shown in the following snippet.

```sh
POST /login HTTP/1.1
Host: fickle-tempest.picoctf.net:59582
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:152.0 Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 20
Origin: http://fickle-tempest.picoctf.net:59582
Connection: keep-alive
Referer: http://fickle-tempest.picoctf.net:59582/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

user=ss&password=sss
```

Nothing is hidden; it is just some HTTP headers.

After that POST request, we get redirected to another page, and the client sends a GET request to `/flag` as follows:

```sh
GET /flag HTTP/1.1
Host: fickle-tempest.picoctf.net:59582
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:152.0 Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: http://fickle-tempest.picoctf.net:59582/
Connection: keep-alive
Cookie: password=sss; username=ss; admin=False
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

The interesting thing here is that the cookie field named `admin` is set to `False`. Let's send this request to the repeater, change it to `True`, and see what happens.

If we change it to `True` and send it, we get logged in as admin, and with that, the flag is shown in the HTML in plain text.

```html
                   <li role="presentation"><a href="/logout" class="btn btn-link pull-right">Sign Out</a>
                    </li>
                </ul>
            </nav>
            <h3 class="text-muted">Factory Login</h3>
        </div>

        <div class="jumbotron">
            <p class="lead"></p>
            <p style="text-align:center; font-size:30px;"><b>Flag</b>: <code>picoCTF{th3_c0nsp1r4cy_l1v3s_FindYOUROWN}</code></p>
        </div>
```

## Conclusion

As I said, it was more of a recon and behavior-understanding type of lab, like all easy picoCTF labs. Two easy labs are left, and after that, Inshallah, we will be starting with medium ones. Hopefully, something interesting is on the way.
