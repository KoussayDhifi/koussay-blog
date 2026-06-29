---
title: PicoCTF No FA Writeup
description: A detailed writeup of the PicoCTF No FA web challenge, covering 2FA bypass and session token manipulation.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, 2FA, SessionHijacking]
pin: false
math: false
mermaid: true
---

## Introduction

This is a medium PicoCTF web challenge titled No FA. The goal is to bypass a weak two-factor authentication implementation and access the admin account.

The challenge provides a login page, a leaked database, and the application source. The weak point is that the OTP is stored in the session instead of being properly secured.

## Recon

The login page asks for a username and password. After testing the default credentials, we find that authentication fails.

![No FA login page](../assets/pico/no-fa/no-fa-1.png)

The HTML form submits a normal POST request to `/login`. The challenge also provides the application source code and a database file.

From the source, we learn that the application stores the OTP in the Flask session:

```python
session['otp_secret'] = otp
session['otp_timestamp'] = time.time()
session['username'] = username
session['logged'] = 'false'
```

This is the weakness we will exploit.

## Extracting the Admin Credentials

The leaked database contains the users table, including the admin account. The admin password hash is present there.

```sql
select * from users;
```

The admin hash is a SHA-256 hash, so it can be cracked with a service like CrackStation. The recovered password is:

```text
apple@123
```

![Cracked password](../assets/pico/no-fa/no-fa-2.png)

## Bypassing the Two-Factor Flow

After logging in with the admin credentials, the application redirects us to the OTP page. The session cookie contains the OTP secret in plain form.

![OTP page](../assets/pico/no-fa/no-fa-3.png)

We can decode the Flask session cookie to recover the OTP value:

```json
{
  "logged": "false",
  "otp_secret": "1149",
  "otp_timestamp": 1781900712.318002,
  "username": "admin"
}
```

With that value, the session can be modified or replayed to bypass the 2FA flow and complete the login.

![Solved lab](../assets/pico/no-fa/no-fa-4.png)

## Conclusion

This challenge demonstrates how weak session handling and insecure 2FA implementation can be abused. The main lesson is that secrets should never be stored in the browser session in a way that can be retrieved and abused.
