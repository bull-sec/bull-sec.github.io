---
layout: post
title: "Cheatsheet: X Without Y"
date: 2022-07-30 23:31 +0100
tags: pentesting windows linux kali cheatsheet
categories: [Cheatsheets]
published: true
---

Inspired by [Staaldraad's Post on the Matter](https://staaldraad.github.io/2017/12/20/netstat-without-netstat/)

Run into situations like this quite often when testing container based systems so thought it was worth starting to compile them all into one document for reference purposes.

## ARP without ARP

```bash
cat /proc/net/arp
ip neigh
```

## Netstat without Netstat

Staaldraad's AWK script:

```bash
awk'function hextodec(str,ret,n,i,k,c){
    ret = 0
    n = length(str)
    for (i = 1; i <= n; i++) {
        c = tolower(substr(str, i, 1))
        k = index("123456789abcdef", c)
        ret = ret * 16 + k
    }
    return ret
}
function getIP(str,ret){
    ret=hextodec(substr(str,index(str,":")-2,2)); 
    for (i=5; i>0; i-=2) {
        ret = ret"."hextodec(substr(str,i,2))
    }
    ret = ret":"hextodec(substr(str,index(str,":")+1,4))
    return ret
}
NR > 1 \{\{if(NR==2)print "Local - Remote";local=getIP($2);remote=getIP($3)}{print local" - "remote}}' /proc/net/tcp
```

Remove the backslashes before the if statement after `NR > 1` to get it to work.

The above presumes `awk` is installed, if it's not you're SOL.

## Ping without Ping

Two cases for this.

- 1) We don't have `ping`
- 2) We do have ping but ICMP is blocked

We can't get around Case #2 but we can get around Case #1.

```bash
telnet ip.ad.dre.ss 80
nc -znv ip.ad.dre.ss port
```

## ls without ls

```bash
echo *
echo */*
for i in *; do echo $i; done
```
