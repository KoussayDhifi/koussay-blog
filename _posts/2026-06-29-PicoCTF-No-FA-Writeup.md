---
title: PicoCTF No FA Writeup
description: A writeup of the PicoCTF No FA challenge with a 2FA session bypass using a leaked database and Flask session cookie.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, 2FA, SessionHijacking]
mermaid: true
---

## Introduction

This is a medium web CTF challenge from PicoCTF titled [No FA](https://learn.cylabacademy.org/library/765?category=1&page=1&difficulty=2).

It has the following description: **Seems like some data has been leaked! Can you get the flag?**

This is my first medium PicoCTF challenge, so let's try it.

## Recon

We are provided with the backend code of the website, along with a database and the actual website. First, we are going to do manual investigation and fill in the unknown pieces with the code so we can have a complete picture.

### Manual Investigation

When we enter the website, we are redirected to the `/login` page, which contains a username field, a password field, and a login button, as shown in the following image.

![No FA login page](../assets/pico/no-fa/no-fa-1.png)

When I try to insert default credentials like `admin:admin`, I get an error message stating `Invalid username or password`.

When I check the HTML code, it is a normal form with a POST request to `/login`. Now let's intercept the traffic with Burp Suite before we check the provided code source for more information.

Basically, the Burp interception is normal and nothing fishy is going on, as shown.

```sh
POST /login HTTP/1.1
Host: foggy-cliff.picoctf.net:53962
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:152.0 Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: http://foggy-cliff.picoctf.net:53962
Connection: keep-alive
Referer: http://foggy-cliff.picoctf.net:53962/login
Upgrade-Insecure-Requests: 1
Priority: u=0, i

username=admin&password=admin&action=
```

The data is sent in the format `application/x-www-form-urlencoded`.

Now we have checked all the manual aspects; let's look at the code source.

### Code Source Investigation

After checking the code source, we found that it is a normal website for logging in, and, of course, the flag is in the admin account. What is interesting is that some accounts are protected with some kind of 2FA. The problem with that 2FA is that it stores the OTP (one-time password) not in the database, but it uses a bad practice that I used to use in my early dev days — storing the confirmation number in the session in the user's browser. So basically anyone with basic Burp knowledge can get this number and bypass the useless 2FA.

```py
if user['two_fa']:
    # Generate OTP
    otp = str(random.randint(1000, 9999))
    session['otp_secret'] = otp  # Here the OTP is stored in the session, so we can retrieve it
    session['otp_timestamp'] = time.time()
    session['username'] = username
    session['logged'] = 'false'
    # send OTP to mail ---
    return redirect(url_for('two_fa'))
```

The problem is where we get the email and passwords. The challenge provided us with the leaked database, and after checking it, we get the following.

```sql
sqlite> .tables
users
sqlite> select * from users
   ...> ;
1|john.doe|john.doe@nfa.com|599a4410e2af69d1585f16d82d4b5f0abf3ad09fa42b9d55d7b7a50671ccf8c1|0
2|jane.smith|jane.smith@nfa.com|81c68634d1b211e0d5632839f7efc8601c743f1ef0c94da8220e26ab221efff1|0
3|robert.jones|robert.jones@nfa.com|aaf120fcb16e20e2d18e63e668e060b5e4a52c5e0b3f038777365fe87ca2ccdb|0
4|emily.brown|emily.brown@nfa.com|9e85668a071a595fe9222725bfb591cdaa0d880e3a7c7de1d9ddd3d4b7d08772|0
5|admin|iamadmin@nfs.com|c20fa16907343eef642d10f0bdb81bf629e6aaf6c906f26eabda079ca9e5ab67|1
6|michael.davis|michael.davis@nfa.com|576454d8921440f30609200a7f79073ec5b69ee284f27bbb860620d56416ad94|0
7|linda.wilson|linda.wilson@nfa.com|082a6006d9c87749adff6be260461171b508744a90a45f75abe78d92995485c5|0
8|david.garcia|david.garcia@nfa.com|faa32a09d4798d21486344a140fd0977cbec33d5b045bca83c04efb364c49d9|0
9|jennifer.rodriguez|jennifer.rodriguez@nfactf.org|c1488b6d9ed8352a64f979506583f33d80aa4119190f7892bc481e8984c880d0|0
10|christopher.williams|christopher.williams@nfa.com|0bf3a14c03e9c7034b9588a69f828840fd32bd739c37b613f41c4aecee26e277|0
11|angela.martinez|angela.martinez@nfa.com|e64b5893827166e4568af8ece105d8c0839772ae10fba3c11e77b5fb3c0ef0c6|0
12|kevin.anderson|kevin.anderson@nfa.com|8bac48021ebd453dbd876d43fa28c8e383fc16176fc8b12fa474b01eb9fa4df5|0
13|melissa.thomas|melissa.thomas@nfa.com|564c89c28d93e8485b76a41deca21ab28e60a32c506e479b925f4643722e9f83|0
14|brian.jackson|brian.jackson@nfa.com|7fccba2f216750414443626058128539ef5a8859f7cb20da2b22d8d787ec6fc2|0
15|stephanie.white|stephanie.white@nfa.com|64acea3bdefef67d65e6a36ee66ac66e85d39931639ea926d1fc98fedd28905b|0
16|eric.harris|eric.harris@nfa.com|b9590eaeaa25401398ebd4b98e10182f4e265f396f23a11eb8fdb18d66a1685c|0
17|michelle.martin|michelle.martin@nfa.com|9b68124e23f3bb700682d28d1d750bec95794a193097b59526ef038f810cb34c|0
18|patrick.thompson|patrick.thompson@nfa.com|1549f62e486c006cbbacee5947c3f6815a0c5f3ef54c80f1f0b17c2ae9da5866|0
19|nicole.garrett|nicole.garrett@nfa.com|5647517c88d64c95170fdb734dc22ba45e284f219d1266eb14f4d9dd7a099ce3|0
20|joseph.cole|joseph.cole@nfa.com|49a57175de704a0ec2a006746d20d375814581bb3552ce0a0b13683426fd232|0
```

What interests us here is line 5 of the admin account, which actually requires 2FA since its value is 1.

What remains is guessing the admin's password. Since it is SHA-256, it can be crackable in some cases, and we are going to use my favorite website for this, [https://crackstation.net/](https://crackstation.net/). It is the best one out there, with ready wordlists and everything already calculated for you, so there is no need for John the Ripper or Hashcat.

After we insert the admin hash, we actually get it, and it is `apple@123`, as shown in the following image.

![Cracked password](../assets/pico/no-fa/no-fa-2.png)

Now we have everything ready; let's open Burp and get the OTP.

## Exploitation

After we log in as `admin:apple@123`, we get redirected to the OTP page.

![OTP page](../assets/pico/no-fa/no-fa-3.png)

In Burp, we get the session encoded `.eJwty0sKgCAQANC7zFpixgI_lwnJSQR_qK2iu9ei7YN3Q6ohsAcLp0uDQUCdbR98dJ4fEm3mtxkzj-lyA0tKk0FUJJeVNKIUcA3uxWX-jvM5FnheKZwcFA.ajWlqA.Vd0J1VK4kzsMShCci8ZNIb9whW8`. All we have to do is decode it now. We can use [https://www.kirsle.net/wizards/flask-session.cgi](https://www.kirsle.net/wizards/flask-session.cgi), and with that we get the full cookie session.

```json
{
    "logged": "false",
    "otp_secret": "1149",
    "otp_timestamp": 1781900712.318002,
    "username": "admin"
}
```

And with that, we insert it before it expires, and the lab is solved.

![Solved lab](../assets/pico/no-fa/no-fa-4.png)

## Conclusion

That was actually an easy one for a medium lab. There was no real exploitation; it was just understanding the logic and checking the cookie session.
