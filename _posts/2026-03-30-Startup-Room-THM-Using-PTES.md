---
title: THM Startup - Full Attack Chain (FTP Misconfiguration to Root via PwnKit)
description: A complete walkthrough of the TryHackMe Startup machine, covering reconnaissance, exploitation via anonymous FTP upload, credential discovery through PCAP analysis, and privilege escalation using PwnKit.
author: Koussay Dhifi
categories: [Cybersecurity, Penetration Testing]
tags: [THM, Boot2Root, FTP, AnonymousLogin, ReverseShell, PCAP, PrivilegeEscalation, PwnKit, CVE-2021-4034, Linux]
pin: true
math: false
mermaid: true
---


## Introduction

This is a boot2root machine within THM named [startup](https://tryhackme.com/room/startup).

If you want the report without seeing me struggle it is [here](https://koussaydhifi.org/assets/reports/SpiceHut_Report.pdf)

The theme of the challenge is the following:

**We are Spice Hut, a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing, and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck!**



## Methodology

For the methodology, we are going to use the classic PTES since this is a pentesting operation.


## Pre-Engagement Interactions

- **Scope:** Since this is a THM room, we are free to test the machine as needed. The goal is to gain full root access.

- **Legal Authorization:** This is a THM room, so we are practicing in a lab environment with full authorization.

- **ROE (Rules of Engagement):** The pentest will be conducted from 11 AM to 14 PM. All methods are allowed except those that may damage the machine.

---

## Intelligence Gathering (Recon)

**Target IP addr:** 10.129.164.12

In this section, we are going to perform reconnaissance on everything this target has to offer. First, we open the website in the browser by entering the IP address.

Usually, I like to start with passive information gathering like OSINT and DNS enumeration. However, since this isn't an active real website, we move directly to active information gathering.

**TIP: IF THE WEBSITE DOES NOT WORK, OPEN `/etc/hosts` ON LINUX AND ADD THE FOLLOWING LINE**
`IP-ADDR startup.thm`


### Web App Check

The web application returns the following page when accessed:

![Web App](../assets/img/startup/Pasted%20image.png)

The application appears not ready, so let's try to guess the tech stack using whatweb, which is a tool that identifies technologies used by web applications.

---

#### Tech Stack

```sh
$ whatweb 10.129.164.12
http://10.129.164.12 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], Email[#], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.164.12], Title[Maintenance]
```

This gives us the version of the web server and the operating system.

Even after using Wappalyzer (a Chrome extension for the same purpose), we get the same information:

![Wappalyzer](../assets/img/startup/Pasted%20image%20(2).png)

---

### Directory Scanning and Enumeration

We use gobuster to enumerate directories of the web application to find exposed resources.

```sh
===============================================================
Gobuster v3.6
...
===============================================================
```

We found one path, which is `/files`, so let's check it:

![Files Directory](../assets/img/startup/Pasted%20image%20(3).png)

We found several interesting files and directories such as `ftp/`, `important.jpg`, and `notice.txt`.

The `important.jpg` is an Among Us meme:

![Important Image](/assets/img/startup/important.jpg)

The `notice.txt` contains:

```
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

This reveals a potential username: **Maya**.

When we check the FTP directory, we find nothing uploaded:

![FTP Directory](../assets/img/startup/Pasted%20image%20(4).png)



### Port Scanning

We use MSF with db_nmap:

```sh
msf6 > db_nmap -sC -sV -O -T4 -A -p- -v 10.129.164.12
```

We found:

* FTP (21) → vsftpd 3.0.3
* SSH (22) → OpenSSH 7.2p2
* HTTP (80) → Apache 2.4.18



### Enum4linux

Enum4linux did not return useful information since unauthorized access is not allowed.



### FTP Enumeration

We attempted brute forcing using Hydra, but no credentials were found.

However, anonymous login works and allows write operations.



### SSH Enumeration

User enumeration and brute force attempts did not return useful results.


## Threat Modeling

After reconnaissance, we identify valuable assets and possible attack paths.

The most valuable asset is FTP access, since we can upload a reverse shell and execute it through the browser.



## Vulnerability Assessment

* Apache → No useful exploit found
* OpenSSH → CVE-2016-6210 (user enumeration)
* OpenSSH → CVE-2016-10009
* FTP → CVE-2011-0762 (DoS, not useful here)



## Exploitation

We upload a PHP reverse shell:

![Payload Upload](../assets/img/startup/Pasted%20image%20(5).png)

After triggering it and setting up a netcat listener:

```sh
nc -nlvp 4444
```

We obtain a shell as `www-data`.



## Post Exploitation

The first flag is found in `recipe.txt`.


### Enumeration

Current user: `www-data`
Target user: `lennie`

We find a file named `incidents` containing a PCAP file `suspicious.pcapng`.



### Privilege Escalation

Analysis of the PCAP reveals the password:

```
c4ntg3t3n0ughsp1c3
```

We use it to SSH into the machine as `lennie`.



### Root Privilege Escalation

We find `pkexec` version 0.105, which is vulnerable to PwnKit (CVE-2021-4034).

We compile the exploit:

```sh
gcc -shared PwnKit.c -o PwnKit -Wl,-e,entry -fPIC
```

After running it, we gain root access:

![Root Access](../assets/img/startup/Pasted%20image%20(6).png)



## Final Flag

```sh
cat /root/root.txt
```

## Reporting

For the full report you can find it (here)(https://koussaydhifi.org/assets/reports/SpiceHut_Report.pdf)

## Conclusion

This was a nice room that started with an FTP misconfiguration, followed by PCAP analysis, and ended with a privilege escalation using PwnKit.


