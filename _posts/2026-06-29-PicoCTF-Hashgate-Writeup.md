---
title: PicoCTF Hashgate Writeup
description: A writeup of the PicoCTF Hashgate challenge, showing how to exploit an IDOR-like profile lookup through MD5-based user IDs.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, IDOR, Enumeration]
pin: false
math: false
mermaid: true
---

## Introduction

This is a medium PicoCTF web challenge titled Hashgate. The application exposes a profile page that is intended to be accessible only to the logged-in user, but the profile ID is derived from a simple MD5 hash of an integer value.

The challenge description hints that the admin profile is accessible through a hidden or obfuscated path.

## Recon

The login page asks for an email and password. The page also includes a comment with some sample credentials:

```html
<!-- Email: guest@picoctf.org Password: guest -->
```

After logging in as the guest user, we are redirected to a profile page.

![Hashgate login page](../assets/pico/hashgate/hashgate-1.png)

The profile URL contains a hashed user ID, which appears to be derived from the numeric user ID.

![Hashgate guest profile](../assets/pico/hashgate/hashgate-2.png)

## Exploitation

The profile lookup is based on an MD5 hash of the numeric ID. If the guest user has ID 3000, the corresponding URL uses the MD5 hash of `3000`.

By enumerating nearby IDs and hashing them, it becomes possible to discover the admin profile URL.

```python
import hashlib

for i in range(3000, 3021):
    url = f"http://target/profile/user/{hashlib.md5(str(i).encode()).hexdigest()}"
    print(url)
```

This reveals the admin profile page for ID 3019.

![Hashgate admin profile](../assets/pico/hashgate/hashgate-3.png)

## Conclusion

This challenge is a great example of IDOR-style access control issues. The application trusted an identifier that was easily guessable and derived from a predictable hash, which allowed the admin profile to be accessed.
