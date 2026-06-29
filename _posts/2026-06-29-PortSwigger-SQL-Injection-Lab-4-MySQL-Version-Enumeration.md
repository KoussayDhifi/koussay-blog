---
title: PortSwigger SQL Injection Lab 4 — MySQL Version Enumeration
description: A writeup of PortSwigger's SQL injection lab for retrieving the MySQL version through UNION-based injection.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, MySQL]
pin: false
math: false
mermaid: true
---

## Introduction

This lab is very similar to the Oracle version lab, but it targets MySQL or Microsoft SQL Server instead.

The objective is to retrieve the database version string by exploiting the category filter.

## Recon

The application presents the same kind of catalog and category filter as in the earlier labs.

![Lab 4 overview](../assets/sqli/lab4/4-1.png)

## Exploitation

We first confirm that the input is vulnerable by submitting a quote or a boolean-based payload.

A payload such as:

```sql
' OR 1=1 -- 
```

is enough to confirm that the application is vulnerable to SQL injection.

Once that is confirmed, we use a UNION attack to reveal the database version. The MySQL query we want to trigger is:

```sql
SELECT @@version;
```

So we craft a payload like:

```sql
' UNION SELECT @@version, NULL -- 
```

This returns the server version in the response, which solves the lab.

![Lab 4 payload](../assets/sqli/lab4/4-2.png)

![Lab 4 solved](../assets/sqli/lab4/4-3.png)

## Conclusion

This lab teaches the same core UNION injection technique as the Oracle version lab but with MySQL-specific syntax. The important idea is still the same: the injection must match the number and types of columns expected by the original query.
