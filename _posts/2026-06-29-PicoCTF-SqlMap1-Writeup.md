---
title: PicoCTF SqlMap1 Writeup
description: A writeup of the PicoCTF SqlMap1 challenge, focusing on SQL injection and password cracking.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, SQLInjection, SQLite]
pin: false
math: false
mermaid: true
---

## Introduction

This PicoCTF challenge is titled SqlMap1. The description hints at sloppy code, legacy hashing, and a hidden SQL injection vulnerability.

The goal is to exploit the search functionality to retrieve user data from the database and then recover the password for a valid account.

## Recon

The application provides a login form and a registration form. Creating a user reveals that the database is SQLite and that the username column is unique.

![SqlMap1 registration](../assets/pico/sqlmap1/sqlmap1-1.png)

The item search functionality is vulnerable. Entering a value like `flag'` triggers a SQL syntax error, which confirms that the input is being interpreted as SQL.

![SqlMap1 injection error](../assets/pico/sqlmap1/sqlmap1-2.png)

## Exploitation

The vulnerable query is likely similar to a `LIKE` search. By injecting union-based SQL, it becomes possible to retrieve data from the `users` table.

A payload such as:

```sql
' UNION SELECT username, password FROM users --
```

reveals the stored usernames and password hashes.

![SqlMap1 user dump](../assets/pico/sqlmap1/sqlmap1-3.png)

The recovered password hash can then be cracked. Once the password is recovered, the corresponding user can be logged in and the flag is revealed.

![SqlMap1 solved](../assets/pico/sqlmap1/sqlmap1-4.png)

## Conclusion

This challenge is a practical example of how SQL injection can lead to credential exposure. It also shows that even a medium challenge can be solved by combining injection with simple password cracking.
