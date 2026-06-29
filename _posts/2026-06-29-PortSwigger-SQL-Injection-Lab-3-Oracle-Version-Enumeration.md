---
title: PortSwigger SQL Injection Lab 3 — Oracle Version Enumeration
description: A writeup of PortSwigger's Oracle-based SQL injection lab for extracting the database version with a UNION attack.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, Oracle]
pin: false
math: false
mermaid: true
---

## Introduction

This lab introduces UNION-based SQL injection. The challenge is to retrieve the Oracle database version string from the application.

The vulnerable parameter is the category filter, and the application is similar to the earlier labs in the series.

## Recon

The site presents the usual product catalog and category selection interface. By interacting with the category filter, we can trigger the vulnerable request.

![Lab 3 overview](../assets/sqli/lab3/3-1.png)

## Detecting the Vulnerability

A single quote in the parameter can trigger an error, which is a strong sign that the input is being interpreted as SQL.

```text
/filter?category=Accessories'
```

This confirms that we are dealing with a SQL injection vector.

![Lab 3 error](../assets/sqli/lab3/3-2.png)

## Exploitation

Since Oracle requires a `FROM` clause in every `SELECT` statement, we can use the `dual` table when crafting our payload.

To determine how many columns the original query returns, we can try:

```sql
' UNION SELECT NULL FROM dual --
```

and then add more `NULL` values until the query stops throwing an error.

Once the column count is known, we can extract the version string with:

```sql
' UNION SELECT BANNER, NULL FROM v$version --
```

That reveals the Oracle database version and solves the lab.

![Lab 3 solved](../assets/sqli/lab3/3-3.png)

## Conclusion

This lab is a good introduction to UNION injection on Oracle. It shows how to identify the number of columns and how Oracle-specific syntax affects the payload.
