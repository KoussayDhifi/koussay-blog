---
title: PortSwigger SQL Injection Lab 1 — Retrieving Hidden Data
description: A writeup of PortSwigger's first SQL injection lab, showing how to retrieve hidden data by injecting into a WHERE clause.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs]
mermaid: true
---

## Introduction

This is the first SQLi lab within PortSwigger titled [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data).

It is a starting point for people to get used to SQLi vulnerabilities and exploitation.

## Recon

For this lab, we have the following e-commerce shopping website that contains four categories, as shown in the following image.

![Lab 1 overview](../assets/sqli/lab1/1-2.png)

When clicking on a category, it executes the following SQL clause.

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1;
```

And our goal is to inject an SQL clause that lists both released and unreleased products.

When we check the category, it goes to the following URL: `filter?category=Clothing%2c+shoes+and+accessories`. Basically, the injection will happen there in the variable `?category`.


## Exploitation and Payload

Since we are injecting in the category and want to list both released and unreleased products, we will write:

```SQL
SELECT * FROM products WHERE category = 'Clothing%2c+shoes+and+accessories' OR '1'='1' -- AND released=1';
```

We will basically be injecting the following payload: `?category=Clothing,+shoes+and+accessories' OR '1' = '1' --`

We will close the category string, and by adding `OR '1'='1'`, we will always have a `True` Boolean result. Since `released = 1` will be commented out, both released and unreleased products will be shown, and the lab will be solved.

![Lab 1 solved](../assets/sqli/lab1/1-1.png)

## Conclusion

That was a nice and simple challenge for a basic introduction to SQLi. We used the famous payload `' OR '1'='1' --`.
