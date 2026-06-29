---
title: PicoCTF Credential Stuffing Writeup
description: A writeup of the PicoCTF Credential Stuffing challenge, showing how to automate login attempts against a TCP-based banking service.
author: Koussay Dhifi
categories: [Writeups, PicoCTF]
tags: [WebExploitation, PicoCTF, CredentialStuffing, Sockets]
pin: false
math: false
mermaid: true
---

## Introduction

This PicoCTF challenge is about credential stuffing. The task is to take a leaked list of credentials and try them against a simple login service until a successful login is found.

The service is reachable over a raw TCP connection, so a small script is enough to automate the attack.

## Recon

The challenge gives us a list of credentials in a text file. The service itself is a simple command-line login prompt.

```sh
nc crystal-peak.picoctf.net 58567
```

The login flow prompts for a username and password and responds with either an authentication failure or a success message.

## Exploitation

The approach is to read the credential dump, connect to the service, send each username and password pair, and look for a successful response.

A basic threaded script can automate this efficiently.

```python
import socket
import threading

with open("creds-dump.txt", "rt") as f:
    creds = f.read().splitlines()
```

The correct credentials are eventually found:

```text
allen:talon
```

The successful response contains the flag:

```text
Welcome allen!
picoCTF{d0nt_r3u5e_cr3d3nt1als_Find_Your_own}
```

## Conclusion

This challenge demonstrates how leaked credential pairs can be abused with automated login attempts. It is a practical reminder that password reuse across services is a serious security risk.
