---
title: PortSwigger SQL Injection Lab 2 — Login Bypass
description: A writeup of PortSwigger's login bypass lab, demonstrating how to bypass authentication with SQL injection.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs]
mermaid: true
---

## Introduction

This is the second SQLi lab on PortSwigger titled [SQL injection vulnerability allowing login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass).

The description is as follows:

**This lab contains a SQL injection vulnerability in the login function. To solve the lab, perform a SQL injection attack that logs in to the application as the administrator user.**

## Recon

When we enter, we go directly to the login page, since that is where the SQLi will occur.

![Lab 2 overview](../assets/sqli/lab2/2-1.png)

## Exploitation and Payload

Let's suppose that the SQL clause that the backend uses is this.

```sql
SELECT * FROM users WHERE username = '' AND password = '';
```

That is mostly how we solve SQLi problems when we do not have the source code or SQL clause. We suppose or try to imagine how the developers would write the SQL clause.

If it was written like that, then we would inject:

1. Write `admin`
2. Close the string with `'`
3. Add `OR '1'='1'`. The SQL clause will always return a row of users, all of them in this case, and user `[0]` will be the one we impersonate.
4. Comment out the password condition with `--`
5. The payload will be `admin' OR '1'='1' --`


![Lab 2 login bypass](../assets/sqli/lab2/2-2.png)

With that, if we insert this in the username field, the lab is solved.

![Lab 2 solved](../assets/sqli/lab2/2-3.png)

## `admin' --`

Some of you, and me, are asking why not `admin' --`. Actually, we do not know whether the user named `admin` exists in the database or not. So if the clause becomes:

```sql
SELECT * FROM users WHERE username='admin' -- ' AND password='';
```

It may return an empty array (0 rows).

So by adding `OR '1'='1'`, we are being safe and forcing the SQL clause to list all users and thus bypass the login.

## `administrator' --`

Actually, now if we insert `administrator' --`, the lab works because `administrator` exists in the database, since the lab guarantees that the user administrator exists.

## Conclusion

That was another easy lab with a usual payload to help us understand SQLi more and get used to it. We had not started the fun yet.