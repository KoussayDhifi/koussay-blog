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

This lab is all about bypassing authentication with SQL injection in a login form.

The backend logic is probably something like:

```sql
SELECT * FROM users WHERE username = '' AND password = '';
```

The goal is to log in as a valid user without knowing the password.

## Recon

When we open the login page, the username and password fields are the obvious injection points.

![Lab 2 overview](../assets/sqli/lab2/2-1.png)

## Exploitation

A simple payload like this works:

```sql
admin' OR '1'='1' --
```

It closes the username string, forces the condition to true, and comments out the rest.

So the query becomes effectively always true, which lets us authenticate.

![Lab 2 login bypass](../assets/sqli/lab2/2-2.png)

Why not just use `admin' --`? Because if the admin user does not exist, the query still fails. Using `OR '1'='1'` is safer and more reliable.

![Lab 2 solved](../assets/sqli/lab2/2-3.png)

## Conclusion

This lab is a straightforward example of login bypass via SQL injection. It shows how a small injection can turn an authentication check into a universal pass.
