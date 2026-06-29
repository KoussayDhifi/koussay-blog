---
title: PortSwigger SQL Injection Lab 2 — Login Bypass
description: A writeup of PortSwigger's login bypass lab, demonstrating how to bypass authentication with SQL injection.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs]
pin: false
math: false
mermaid: true
---

## Introduction

This lab focuses on bypassing authentication through a SQL injection vulnerability in the login form.

The backend logic is roughly the same as:

```sql
SELECT * FROM users WHERE username = '' AND password = '';
```

The goal is to log in as the administrator without knowing the password.

## Recon

When the login page is opened, the username and password fields are the obvious injection points. The vulnerable logic is likely built around the username and password values supplied by the user.

![Lab 2 overview](../assets/sqli/lab2/2-1.png)

## Exploitation

A simple payload such as:

```sql
admin' OR '1'='1' --
```

works because it closes the username string, forces the query to evaluate as true, and comments out the rest of the condition.

This causes the query to behave as if the login check always succeeds, which lets us authenticate as a valid user.

![Lab 2 login bypass](../assets/sqli/lab2/2-2.png)

A common question is why not use `admin' --` alone. The answer is that the database may not contain a user named `admin`, so the safer approach is to force the condition to return true with `OR '1'='1'`.

![Lab 2 solved](../assets/sqli/lab2/2-3.png)

## Conclusion

This lab reinforces the standard SQL injection pattern for authentication bypass. It shows how a simple injection can turn a normal login check into a universal success condition.
