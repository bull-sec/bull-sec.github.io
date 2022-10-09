# Bart HTB

```
nmap -sC -sV --top-ports 10000 -vvv -O 10.10.10.86 -oA Bart
```

As the scan is running we see port 80, so let's go check it out. 

And a quick check of the results once the scan is finished confirms that the only port open on TCP is 80.

Nothing on UDP either, looks like we're going IIS exploiting. 

## IIS Smashing

Soon as we try to get to the main landing page we're hit with an error:

```
Server not found

Firefox can't find the server at forum.bart.htb
```

We didn't ask to go there?

What if we add it to our host file?

OK, that now works, and dissapointingly Bart doesn't refer to The Simpsons, it's some sort of lame business website... *yawn*

`gobuster` and `wfuzz` are out on the main page, as it redirects all failed requests to a static page that serves a `200`

I *know* this box. I've seen it before and I'm sure I managed to get a shell on it. 

Let's run `cewl` on the main page and scrape a bunch of data, see if we can't turn it into information.

(WHAT VERSION IS IIS RUNNING YOU SPASTIC?!)

Good question voice in my head...

`IIS 10.0 - Which means we're either dealing with Server 2016, or Windows 10`

Cool.

We probably wont' know that until we get onto the box, but we'll keep it in the back of our head for now. 

## Scraping with Cewl

`cewl` is an awesome little tool for scraping data from a webpage.

```
cewl -w cewl_list -e -d 2 http://forum.bart.htb
```

Not much to it really, if you're stuck run `cewl --help` first. 

- `-e` includes emails
- `-w` is the file we want to write out to 
- `-d` is the depth to go looking if it finds links

OK, so we have some emails, which are potentially usernames as well. We have a bunch of words and all kinds of other nonsense. 

```
Email addresses found
---------------------
Mailinfo@bart.htb
d.simmons@bart.htb
info@bart.htb
r.hilton@bart.htb
s.brown@bart.local
```

Interestingly, `cewl` didn't pick up on any of the stuff that had been commented out, although it did pick up some bits of the forthcoming information.

Looks like there is a potential ex employee that they've commented out of the site.

```
Harvey Potter
Developer@BART
h.potter@bart.htb
```

(Edited for clarity)

## Back to IIS

Thinking this through, there are probably some sub-domains with more stuff to look at.

So let's modify our hosts file to have `bart.htb` as a TLD and see if we can't find some of them:

- We know there is a developer, because we've seen his picture so `dev.bart.htb` is a shout
- We know they have to update this page somehow, so there is probably an `admin.bart.htb` 

We'll come up with some more as well go, let's try those out for now. 

None of those seem to work. 

How do we brutefore subdomains when we just get bounced back to the main site each time, and the main site gives us 200's for every page we attempt to visit because the error page is poorly configured?

```
$ wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hh 158607 --hc 404,500 http://bart.htb/FUZZ                                                                       
```

The answer is we don't. We were overthinking it, as usual. 

Using `--hh` to hide that main page from being displayed at us is the key to success in this case!

Because the `bart.htb` page redirected to `forum.bart.htb` whenever the directory at the end didn't point to a valid domain we got a lot of the same results in our `wfuzz`. Using this bit of information we can hide those results until we hit something that actually resolves. 

```
--hc/hl/hw/hh N[,N]+        : Hide responses with the specified code/lines/words/chars (Use BBB for taking values from baseline)        
```

In our case we hide the `chars` that we were seeing. 

We'll leave `wfuzz` running and see what it comes up with at the end.

Not much else:

```
$ wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hh 158607 --hc 404,500 http://bart.htb/FUZZ                                                                       

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.                                   

********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://bart.htb/FUZZ
Total requests: 220560

==================================================================
ID      Response   Lines      Word         Chars          Payload
==================================================================

000001:  C=302      0 L        0 W            0 Ch        "# directory-list-2.3-medium.txt"
000002:  C=302      0 L        0 W            0 Ch        "#"
000003:  C=302      0 L        0 W            0 Ch        "# Copyright 2007 James Fisher"
000067:  C=301      1 L       10 W          145 Ch        "forum"
001614:  C=301      1 L       10 W          147 Ch        "monitor"
002385:  C=301      1 L       10 W          145 Ch        "Forum"
```

Landing on `bart.htb/monitor` we have a login page. Looks like it just uses Basic Authentication, and trying a few different random usernames and passwords doesn't seem to lock us out, so let's script up a hydra run and see if we can get a password. 

We'll chop the emails off the bottom of our `cewl_list` and use that as a basis for our word list for both usernames and passwords, including the extra bits of information we found. 

## Creating the Word List

OK so we have a wordlist we made with `cewl`, we trimmed the Email header off but now we're left with a lot of... shit, to put it bluntly. Let's get rid of a bunch of it. All the lorem ipsum stuff can go, and the buzzwords. 

I put on Van McCoy's "The Hustle" and set to work:

```
BART
info
Samantha
Robert
Developer
Brown
CEO
Daniel
Simmons
Sales
Hilton
Harvey
Potter
Development
FluffyUnicorns
developers
Recruitment
Daniella
Lamborghini
bart
htb
Mailinfo
d.simmons
info
r.hilton
s.brown
h.potter
```
Probably could have done some fancy stuff with `sed`, but the tunes were going strong, and I was feeling the repeat action key in `vim`. 


## Hackin' an Crackin' 

Before we do that we just need to see what the request looks like in Burpsuite. 

```
POST /monitor/ HTTP/1.1
Host: bart.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://bart.htb/monitor/
Cookie: PHPSESSID=989g1h8q69u2ujofuesb7mo96h
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 116

csrf=1526c861227699108f9a1411616359fc2a35d8ab1558f8f4a25e924a1e128a40&user_name=balh&user_password=blah&action=login
```

Sick, so it's a post form, although that CSRF token looks like it could be a problem...

Does it change with each request? 

Thankfully, no it doesn't appear to change. Dodged a bullet there. 

`hydra http://bart.htb/monitor -f -V -L user_list -P user_list http-post-form "/monitor/:csrf=1526c861227699108f9a1411616359fc2a35d8ab1558f8f4a25e924a1e128a40&name=^USER^&password=^PASS^:The information is incorrect."`

Hmmm, `hydra` is throwing up some funky results, and I suspect it may have something to do with the rendering on the page. If you try and highlight something you don't appear to be able to, something in the way the page is being rendered is preventing `hydra` from seeing the error message, even though we can see it in the page and the Burp response. 

OK, since Burp can see it, let's use Burp to crack the password!

## INTRUDER ALERT!

Smashing our word list against this thing for about 30 minutes got me nowhere, so let's even the odds, thanks to hint we can use the "Forgot Password" page to try and enumerate some usernames! :D 

Using that same list in Sniper mode we can quickly enumerate a few usernames. 

The reason we can tell that we found these is simply due to the differing content lengths in Burpsuite. The longer responses contained a message about a password reset being sent to an email address. :thumbs_up: 

- `Daniel`
- `Harvey`

OK, so now we know who to target.

```
Daniel
daniel
Harvey
harvey
Daniel@bart.htb
daniel@bart.htb
Harvey@bart.htb
harvey@bart.htb
```

OK, but what about last names? And modifications? 

```
Harvey
Potter
h.potter
HarveyPotter
H.Potter
harveypotter
Harvey@bart.htb
Potter@bart.htb
h.potter@bart.htb
HarveyPotter@bart.htb
H.Potter@bart.htb
harveypotter@bart.htb

Daniel
Simmons
d.simmons
DanielSimmons


