---
title: PicoCTF SqlMap1 Writeup
description: A writeup of the PicoCTF SqlMap1 challenge using SQL injection and password cracking.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, SQLInjection, SQLite]
pin: false
math: false
mermaid: true
---

## Introduction

This is another PicoCTF medium web challenge that is titled [Sql Map1](https://learn.cylabacademy.org/library/729?category=1&page=1&difficulty=2).

It has the following description of: **You’ve been hired by a shadowy group of pentesters who love a good puzzle. The system looks ordinary, but appearances lie. Somewhere inside, sloppy code and legacy hashing practices left a tiny, perfect doorway for an attacker.
Your mission — should you choose to accept it — is to slip through that doorway, act as a legit user and retrieve the secret flag.**

So the important thing about this description is.

1. Sloppy Code - There may be some logical errors -
2. Legacy Hashing Practices

So the challenge name being SQLMAP1 I think that we will exploit a sql injection vuln to extract user passwords and that password is hashed with some legacy algorithm let's say like SHA-1 and we gonna crack it that's what I'm thinking.

## Recon

So we are faced with a login form as shown in the following image.

![SqlMap1 registration](../assets/pico/sqlmap1/sqlmap1-1.png)

We try sojme trivial creds like `admin:admin` and we find that either the username or password is wrong.

We try to create an account with the name `admin` to see if it accepts it or not and we find something interesting after that which is a sqlite error that gets generated when we insert admin ofcourse with the note saying that this username is already taken.

![SqlMap1 invalid username](../assets/pico/sqlmap1/sqlmap1-2.png)

The error is as follows

```sql
Warning: SQLite3Stmt::execute(): Unable to execute statement: 1, near "'%'": syntax error in /var/www/html/vuln.php on line 39
```

The error says that the constraint UNIQUE has failed because we inserted a non unique username that already exists in the database.

When we create a normal account let's say with the credentials `koussay:123` we are faced with the following page that tells us to fetch any item we want.

If we type flag there we get a list of course of non valid flags as shown in the following image.

![SqlMap1 search page](../assets/pico/sqlmap1/sqlmap1-3.png)

## Vulnerability Detection and Analysis

Let's try to mess up a little bit since I saw the sqllite error from before I'll try to just insert `flag'` in the items field and we got the following error.

```sql
Warning: SQLite3::query(): Unable to prepare statement: 1, near "'%'": syntax error in /var/www/html/vuln.php on line 39

Fatal error: Uncaught Error: Call to a member function fetchArray() on bool in /var/www/html/vuln.php:40 Stack trace: #0 {main} thrown in /var/www/html/vuln.php:40
```

So from what I understand from the error the query may contain something like `WHERE item LIKE '%USERINPUT%'` because the error mentions that there is a syntax error near `'%'` So we need to inject a payload that knows how to handle the '%' basically this screams SQLinjection because our user input is not treated as text but as sql code and the evidence is that we generated a sql error from our messy input.

## Payload and Exploitation

First we need to know how many columns the first sql select clause returns because we are going to perform a `UNION` attack my assumption is that it returns 2 columns because when we search for flag initially 2 items each line is shown flag label and its value so let's try to inject `' UNION SELECT NULL,NULL -- ` and we eventually get a valid result as shown in the following image.

![SqlMap1 union select](../assets/pico/sqlmap1/sqlmap1-4.png)

So let's try to inject the payload `' UNION SELECT username,password from users -- ` as just a test since previous error showed that there is a table named `users` it may work and we won't waste much time doing recon to the database.

and evetually a list of usernames and hashed password - JUST LIKE I PREDICTED - are shown within the HTML as shown in the following image.

![SqlMap1 user dump](../assets/pico/sqlmap1/sqlmap1-5.png)

The admin credentials are `admin: 5a9a79d9fa477ed163b89088681672c9` so let's use crackstation to see if this hash is weak or not - spoilers it is inshallah -

If we try to crack it we find that it is not found... Weird let's try other user's password like `ctf-player` and evetually we get its plain password which is the following.

![SqlMap1 cracked password](../assets/pico/sqlmap1/sqlmap1-6.png)

And those are the credentials `ctf-player:dyesebel` and after we login we find the flag as shown in the following image.

![SqlMap1 solved](../assets/pico/sqlmap1/sqlmap1-7.png)

## Conclusion

That was an easy challenge rather than a medium one. I mean easy medium as always to medium PicoCTF challenges.
