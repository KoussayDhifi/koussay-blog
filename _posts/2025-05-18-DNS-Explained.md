---
title: DNS Explained
description: This article delves into the DNS protocol what is it  and how it works.
author: Koussay Dhifi
categories: [Networking, DNS, Information Gathering]
tags: [Networking, DNS, DNS zone transfer]
pin: true
math: true
mermaid: true
---

## Introduction
In the realm of networking and cybersecurity, DNS is one of the fundamental pillars that keeps the internet running.

It plays a crucial role of translating human-friendly domain names to IP addresses.

This article explores how DNS works and the structure of its records.


## Domain Name System

In the browser when we type **google.com** the browser and the network don't actually understand this human-friendly text. They only understand the IP notation whether it is a decimal-dotted one like *IPv4* or a hexadecimal one like *IPv6*.

This is where DNS comes into play.

DNS -Domain Name System- is an application-layer protocol that translates or resolves domain names to IP addresses.

## Domain Name Space

The domain name space is the hierarchical structure of a domain name, to simplify it is the set of compositions of the domain name, the following figure shows the domain name space of the *www.google.com* domain name.

![Google DNS Namespace](../assets/img/dns/www.google.com.png)

You may wonder about the last dot in the domain **www.google.com.**. By default every domain name ends with a dot, if it is not inserted by the user the browser will automatically do it instead. You can try it yourself by adding a dot in every domain name you want to resolve and it will work.

Each level in the domain name hierarchy is referred as **domain name zone**, in other words the **.com** is considered a domain name zone, the **google** and so on.


## DNS Actors

Before we get to know how the DNS works, we need to explore the different actors of the DNS protocol. Since the domain names don't get resolved to IP addresses by themselves.


### Recursive Resolver

The first actor in the DNS resolution process is the **Recursive Resolver** which is basically a server that resolves the IP address in response to a query from the browser.

The Recursive Resolver acts as a proxy or an intermediary between the client (the browser) and the other actors that participate within the DNS resolution process.

### Authoritative Name Server

It is the server that stores the file (called : **zone file**) which contains all the information about a specific domain name including its IP address. In fact this server is the one that provides the resolved IP address to the recursive resolver.


### Top Level Server

This is the server that governs everything related to the top level domain (e.g **.com**, **.io** ...).

This server indicates to the recursive resolver the IP address of the **authoritative name server**. To simplify, it points the resolver to the suitable authoritative name server.


### Root Server

The root server is responsible for governing the root domain name. 

One of its roles is to receive a query from the recursive resolver and sends a response indicating the suitable **top level domain server** for the domain name


## DNS Resolution

After diving into the fundamental pillars of the DNS protocol. This section dives into the process of resolving the domain name to an IP address.

The resolution is more like `I know a guy who knows a guy` type of thing and it is summarized into five steps.

The following video visually explains the steps of the dns resolution.

<div style="text-align:center;">
    <video width="640" height="360" controls autoplay>
    <source src="/assets/vid/DNSResolutionWithCenteredText.mp4" type="video/mp4">
    Your browser does not support the video tag.
    </video>
</div>


### Step One : Browser queries recursive resolver

When the user types **www.example.com** in the browser, it either gets the IP address of the website from the browser's cache if it's one of the previously visited websites or it gets the IP address from the ISP's cache.

In our case we will suppose that this is a website that has never ever been visited. In that case the browser sends a query to the recursive resolver asking for the IP address of that specific domain name.


### Step Two : Recursive resolver queries the root server

In this step the recursive resolver sends a query to the root server, the root server will send in response the suitable TLD (Top level domain) server.


### Step Three : Recursive resolver queries the top level domain server

The recursive resolver queries the top level domain server to find the last destination in response which is the suitable authoritative name server.

### Step Four : Recursive resolver queries the authoritative name server

After this query the authoritative name server returns the IP address from the zone file and the recursive resolver finally gets it.

### Step Five : Recursive resolver returns the resolved IP address to the browser

In this step the recursive resolver sends the response that the browser has been waiting for throughout this whole process which is the IP address of the website that is trying to access.

## True Role Of DNS

The resolution of a domain name to its IP address is actually one of the roles of the DNS protocol, but it's definitely not the only one.

In this article something crucial is mentioned which is the **zone file**, this file doesn't only contain the IP address of the domain name, but think of it as the phone book of the whole website, it contains the set of other crucial fields aside from the IP address, where each field is referred as **record**.

So aside from resolving IP addresses, the DNS protocol also resolves other variables by checking the dns records stored in the zone file. Next we will dive into these records and what each type of record resolver.

## DNS Records

DNS records are a pieces of information stored in the authoratative name server, more specifically in the zone file. It stores about everything related to a domain name.

Almost each DNS record has this structure


| Name | Type | Value | TTL (Time to live) |


- **Name** : The key, in other words what we want to resolve
- **Type** : Type of record
- **Value** : The resolved value
- **TTL** : How long the record is cached by the browser, it is measured in seconds.

### A record

This is a type of record that maps the domain name to an IPv4 address.

It is the record used in the resolution of domain names to IP addresses (The previous example).

The following is an example of an A record.

| Name | Type | Value | TTL (Time to live) |
|------|------|-------|--------------------|
| example.com | A | 10.10.12.11 | 3600 |

### AAAA record (Quad A record)

This record maps the domain name to an IPv6 address.

Same as the A record but for IPv6.

The following is an example of a quad A record.

| Name | Type | Value | TTL (Time to live) |
|------|------|-------|--------------------|
| example.com | AAAA | 2001:0db8:85a3:0000:0000:8a2e:0370:7334 | 3600 |

### CNAME record

Canonical name record maps subdomains or domain names to another domain name.

The following is an example of CNAME record

| Name | Type | Value | TTL (Time to live) |
|------|------|-------|--------------------|
| wwww.example.com | CNAME | example.com | 3600 |
| academy.example.com | CNAME | example.com | 3600 |
| example2.com | CNAME | example.com | 3600 |


### MX record

Mail exchange record points emails to mail exchange servers.

SMTP (Simple mail transfer protocol) uses these records to route emails to their proper host.

There are usually more than one MX record each one has a new column referred as priority (the lower the number of priority the higher it is)

The following is an example of MX records.


| Type | Name         | Priority | Mail Server               | TTL        |
|------|--------------|----------|---------------------------|------------|
| MX   | example.com. | 10       | mail1.example.com.        | 3600       |
| MX   | example.com. | 20       | mail2.backupmail.com.     | 3600       |


### NS record

Name server record list the different authoritative name servers of the domain name. Since the zone file is not stored in one authoritative name server to prevent one point of failure.

| Type | Name         | TTL  | Class | Name Server             |
|------|--------------|------|-------|--------------------------|
| NS   | example.com. | 3600 | IN    | ns1.exampledns.com.     |
| NS   | example.com. | 3600 | IN    | ns2.exampledns.com.     |


**NOTE**: The class column to indicate which protocols are being used, the `IN` stands for internet protocols, almost every single record has the class value of IN since they resolve the domain names using the common internet protocols we know about. There are some other legacy values like `CH` (Chasonet only used in MIT labs) that are not being used.

### TXT record

This record stores miscellaneous text that contains details about the domain name.

The following is an example of TXT record.


| Type | Name           | TTL  | Class | Value                                                     |
|------|----------------|------|-------|-----------------------------------------------------------|
| TXT  | example.com.   | 3600 | IN    | "v=spf1 include:_spf.google.com ~all"                     |
| TXT  | _dmarc.example.com. | 3600 | IN | "v=DMARC1; p=none; rua=mailto:dmarc-reports@example.com" |


### SOA record

State of authority record, contains metadata about a DNS zone file.

the following is an example of SOA records.


| Field             | Value                       | Description                                                 |
|------------------|-----------------------------|-------------------------------------------------------------|
| Name             | example.com.                | The root domain of the zone.                                |
| TTL              | 3600                        | Time To Live in seconds.                                    |
| Class            | IN                          | Internet class.                                             |
| Type             | SOA                         | Record type.                                                |
| Primary NS       | ns1.exampledns.com.         | Primary authoritative name server for this zone.            |
| Responsible Email| admin.example.com.          | Email of domain admin (written with `.` instead of `@`).    |
| Serial           | 2025052601                  | Serial number for the zone file (YYYYMMDDnn format).        |
| Refresh          | 7200                        | Time before secondary servers check for updates (in sec).   |
| Retry            | 1800                        | Time to wait if refresh fails (in sec).                     |
| Expire           | 1209600                     | Time before secondary stops serving if no refresh (in sec). |
| Minimum TTL      | 86400                       | Default TTL for negative responses (NXDOMAIN).              |

### PTR record

This is the reverse of A and AAAA records, it maps IP addresses to domain names.

The following is an example of PTR record.


| Type | Name                 | TTL  | Class | Points To            |
|------|----------------------|------|-------|----------------------|
| PTR  | 1.0.0.127.in-addr.arpa. | 3600 | IN    | localhost.example.com.|
| PTR  | 4.3.3.7.0.7.3.0.e.2.a.8.0.0.0.0.0.0.0.0.3.a.5.8.8.b.d.0.1.0.0.2.ip6.arpa.                    | 3600 | IN    | host.example.com.   |



**NOTE**: In PTR record the IPv4 and IPv6 are reversed in the shown way, with `.in-addr.arpa.` suffix for IPv4 and `.ipv6.arpa.` suffix in IPv6.


### SRV record

This record indicates which service is stored at a specific port number.

| Type | Name                      | TTL  | Class | Priority | Weight | Port | Target                 |
|-------|---------------------------|------|-------|----------|--------|------|------------------------|
| SRV   | _sip._tcp.example.com.    | 3600 | IN    | 10       | 60     | 5060 | sipserver.example.com. |
| SRV   | _sip._tcp.example.com.| 3600 | IN    | 10       | 50     | 5060 | sipserver1.example.com.|


**NOTE**: Weight is like a second priority in case of two records have the same priority, in other terms priority determines which group of services to use and weight determines which specific service to use in that group. The higher the weight the heigher its priority among records with the same priority.


## Conclusion

In this article we dived into how DNS works and got to know a crucial and important aspect of the DNS protocol which is the DNS records. By exposing that the DNS doesn't only have the role of resolving domain names to IP addresses but it has many other roles.
So basically we got to know the phone book of the whole internet.

However there are some vulnerabilities in a DNS name server that if an admin or a systems engineer misconfigures the server they can be used to expose crucial information about the website. In the next articles we will dive into DNS zone transer.