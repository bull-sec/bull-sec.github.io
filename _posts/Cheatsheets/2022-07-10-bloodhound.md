---
layout: post
title: "Cheatsheets: BloodHound"
date: 2022-07-30 23:32 +0100
tags: pentesting windows linux kali cheatsheet
categories: [Cheatsheets, Windows]
published: false
---

BloodHound is a tool for visualising and analyzing Active Directory networks so that you can better identify misconfigurations and paths which would allow you to take full control of the Domain Controller (which is the ultimate goal in any Active Directory engagement)

It uses Neo4j to draw interactive graphs which show the links between the objects and Organizational Units within the Domain, it's easier to show than it is to explain:

![BloodHound Demo Image](/assets/img/bloodhound_demo.png)

Each one of those nodes and paths can be interacted with for more information. You can mark each node as "owned" or "high value" depending on what you're looking for. And using some of the prebuilt data queries (the one above is showing "Shortest Path to Domain Admin") you can easily see the route you need to take to elevate your privileges and permissions within an AD network.

> Fun fact: Blood Hound in German is Blut Hund, which appears to be the root of the English version.
{: .prompt-info }

## Neo4j Docker

Getting Neo4j installed feels like a chore, so we can just make use of a docker image instead:

```bash
docker pull neo4j:latest
docker run -p 7474:7474 -p 7687:7687 -d --env NEO4J_AUTH=neo4j/test neo4j:latest
```

We only really need those two ports for BloodHound to be able to communicate with so those two commands will do. If you want a more secure password, change the value of `NEO4J_AUTH=neo4j/test` to something else.

## Installing BloodHound

So I don't actually recommend building this thing, instead I recommend using the Release builds from Github which are kept in line with the `master` branch of BloodHound.

[BloodHound Github Release Page](https://github.com/BloodHoundAD/BloodHound/releases)

Grab the latest one from there, I'm currently using 4.2.0 (#blazeit) but you're likely reading this at some unspecified point in the future, so use whatever is the latest.

Running it is straightfowards:

```bash
# Assuming you've downloaded the release zip already
7z x BloodHound-linux-x64.zip
cd BloodHound-linux-x64/
./BloodHound --no-sandbox
```

## Getting Data

BloodHound data doesn't grow on trees, we're going to have to use one of the kindly provided "Collectors" to do that for us.

![](/assets/img/2022-10-08-00-40-10.png)

My preferred choice is the "SharpHound.ps1" script as I can load that in remotely using an IEX download cradle but it's dealers choice, both usually work a treat although you're going to want a fairly stable TTY in order to run them.

Another good choice is the good old impacket-smbserver.