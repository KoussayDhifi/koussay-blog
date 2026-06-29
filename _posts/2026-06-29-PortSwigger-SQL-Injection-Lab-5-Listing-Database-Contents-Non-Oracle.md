---
title: PortSwigger SQL Injection Lab 5 — Listing Database Contents on Non-Oracle Databases
description: A writeup of PortSwigger's SQL injection lab for enumerating database tables and columns on non-Oracle systems.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, DatabaseEnumeration]
pin: false
math: false
mermaid: true
---

## Introduction

This lab teaches how to enumerate database contents on a non-Oracle database. The goal is to discover a table that stores user credentials and then retrieve them.

## Recon

The app is the familiar e-commerce site with product categories and a filter parameter. The vulnerable request is the category filter in the URL.

![Lab 5 overview](../assets/sqli/lab5/5-1.png)

## Exploitation

First, we confirm the vulnerability by using a boolean-based payload:

```sql
' OR '1'='1' -- 
```

This causes the application to return products from all categories, proving the injection.

Next, we use UNION to query the information schema and enumerate tables:

```sql
' UNION SELECT table_name, NULL FROM information_schema.tables --
```

That reveals a suspicious table name, and we can then inspect its columns:

```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name = 'users_olfmov' --
```

Finally, we retrieve the relevant values from the table and obtain the credentials needed to solve the lab.

![Lab 5 table enumeration](../assets/sqli/lab5/5-2.png)

![Lab 5 credentials](../assets/sqli/lab5/5-3.png)

## Conclusion

This lab is a great introduction to database enumeration. It shows how UNION injection can be used to move from a simple vulnerability to the discovery of stored usernames and passwords.
