---
title: Token Impersonation Within Windows
description: Token Impersonation is an important topic within windows priv escalation this article explains it.
author: Koussay Dhifi
categories: [Post-Exploitation, Privilege Escalation, Windows, MSF]
tags: [Windows, Impersonation, Incognito]
pin: false
math: true
mermaid: true
---

## Introduction

As we know after initial foothold within a system we move to an important phase called **Post-Exploitation**. This phase has several steps and one of these steps is **Privilege Escalation** and one of the techniques that we use within privilege escalation is **Token Impersonation** This technique is only for windows systems since it uses access tokens. In this article we will delve into this topic from what are tokens and how impersonating them is dangerous to the system and how to impersonate them using **Incognito Module** within **Meterpreter**.

## Access Tokens

According to **Microsoft** an **Access Token** is an object that describes the **security context** of a process or a thread.

In other words it describes the identiy of the user associated with the process and what the user is allowed to do.

`Note: Security Context means identity and privileges`

So whenever a user is logged in it is assigned an access token so he can perform privileged operations.

## How an Access Token is Generated

There are two main processes that play a crucial role in generating an access token and passing it to other processes and these processes are **winlogon.exe** and **userinit.exe**

### winlogon.exe

When a user logs in winlogon.exe generates an Access Token pertinent to that sepcific user.

### userinit.exe

When the access token is generated it is passed to userinit.exe, this process is responsible for creating other processes so when other processes are created using this process they will inherit the security context of the user.



The following animation explains how this operation wokrs.

<div style="text-align:center;">
    <video width="640" height="360" controls autoplay>
    <source src="/assets/vid/accessTokens/DynamicTokenInheritance.mp4" type="video/mp4">
    Your browser does not support the video tag.
    </video>
</div>

## What Does The Access Token Contain

The access token contains both the identity and the privileges of the user associated with the thread of the current process. It is a way for the operating system to indicate that process X is running with these privileges.



