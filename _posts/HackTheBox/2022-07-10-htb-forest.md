---
layout: post
title: "HTB: Forest"
date: 2022-08-23 18:07 +0100
tags: hackthebox htb activedirectory hacking ldap bloodhound
categories: [Pentesting, HackTheBox]
published: true
---

Forest is a fully Windows based machine which is semi-unique on HackTheBox in that it doesn't have a web server running. We start by enumerating the open ports and find LDAP running. From there we're able to use APRep Roasting to get a hash which we crack using `JohnTheRipper` and gain access to the machine using `evil-winrm` where we're able to gather some information to dump into `BloodHound` and from there gain full admin.

Strap in.

![Forest](/assets/img/Forest.png)

## Enumeration

## User.txt

## BloodHound

## Root.txt
