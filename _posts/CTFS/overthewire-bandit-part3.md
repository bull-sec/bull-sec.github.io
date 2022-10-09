---
layout: post
title: "CTF: Bandit - Part 1"
date: 2022-07-30 23:32 +0100
tags: pentesting linux kali overthewire
categories: [CTF, OverTheWire]
published: false
---

## OverTheWire - Bandit - Levels 20-Finish

## Bandit 20

Ahh the difficulty curve is back, after the last one I thought they were starting to go easy on us. Not so with this one. We're been given the hint that we probably need to use `screen` or `tmux` to complete this challenge. `screen` and `tmux` are terminal multiplexers, which is a fancy way of saying you can have multiple terminals open on the same connection. I'll be using `screen` for the purposes of this walkthrough, as I'm not familiar with `tmux`.

Some basic `screen` commands:

```bash
Ctrl + A, C     - Create a new window
Ctrl + A , (n)  - Move between active windows, where (n) is the number ID of the window (0-99)
Ctrl + C        - Exit
```

[This guide](https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/) also has some good tips. And the `man` pages are pretty detailed, so you should be able to pick it up pretty quickly (if you've gotten this far, what's one more tool?)

Disaster! Unfortunately for me, `screen` didn't seem to work (it was moaning about file permissions)

That meant learning `tmux` on the fly... Luckily, it turns out that `tmux` is pretty dope, and not too dissimilar from `screen`.

```bash
Ctrl + C    - Create a new Terminal 
Ctrl + (n)  - Move to Terminal (n)
```

(To quit `tmux` just type `exit` into your terminal windows (however many you spawned))

So, what do we actually have to do here?

Actually it's pretty simple. Fire up at least one more Terminal window (you're going to need it) and run the SUID binary that lurks in the home directory. Check out how it works.

```bash
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

Awesome, so we give it a port number, then we give it a password, and it poops out the password for the next level. Nice.

To do that, we're going to need `netcat`. In one of your terminal windows run the following command (substituting the port for whatever you like):

`nc -l 6969 < /etc/bandit_pass/bandit20`

This command will create a listener on port `6969` and direct the contents of `/etc/bandit_pass/bandit20` into it (the password for the current level).

Then flip back to your spare terminal (the one that isn't running `netcat`) and run the SUID binary with your specified port, and you should get a response back from the server with the password for the next level.

## Level 21

OK, confession time. I stopped doing this for ages because I got consumed by University work, and then I got a job. Now I'm going back to it after working in CyberSec professionally for the past 6 months, and realising that (A: I've learned a lot, and B: I actually knew a lot back when I started this, which I believe was the back end of 2016)

Anyway, back to the challenges!

Time for some `cron`! :D (get it? cos cron is a time based thing? OK, I'll just be over here giggling) Terrible jokes aside, we're being asked to check out a scheduled task that's running on this machine using the `cron` application.

To do this we're only going to need to enlist the help of our two trustiest tools, `ls` and `cat`.

Run an `ls` command on the path we're give in the specification for the level:

`ls /etc/cron.d/`

That will list the files in the folder, then we simply need to `cat` them out and check out the command that they're running.

Luckily for us, the first one `bandit22` is the one we're looking for. Check out the command it's running, then head on over to that location and `cat` the contents of it out (don't assume, like I did, that the file name is the password, even though the length matches).

Got it?

Awesome, let's crack on! We're close to the finish line, not much left to do!

## Level 22

Woop! We get to actually read some code! (I like code)

Following on from the last challenge, we've got another `cron` job on our hands (and it doesn't take a genius to note that there was one more file in the `/etc/cron.d/` folder once we're done with this one). If we just repeat the process from last time (i.e. cat out the contents of the `cron` command) we're given something rather interesting.

```bash
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

It's a bash script. Specifically a bash script that does some stuff to the `whoami` command and then uses that to create a directory. Let's run that `mytarget` command locally, see what it's actually doing.

`whoami | md5sum | cut -d ' ' -f 1`

OK, cool, so what does the script do next?

Well, it takes it's newly created MD5, and uses that as the name for a file in the `/tmp/` directory. So, what happens if we run this command, then we cat the contents of `/tmp/MD5GoesHere`?

Duh, I mised a huge part of this...

Note the start of the string:

`echo I am user $myname | md5sum | cut -d ' ' -f 1`

We're doing an `echo` first, somehow my brain ignored that pretty important factor, so now when we try to cat the contents of `/tmp/MD5GoesHere` we get:

A failure. Because we're not running as `bandit23` we're running as `bandit22`... But, all is not lost, because we can just replce the value of `$myname` with `bandit23`.

Now if you run it again, then copy the output string and then do a cat on `/tmp/<string what you found>`, you should get the password to the next level.

## Level 23

It's the final countdown! (aka the last `cron` challenge)

Now things are really starting to get interesting. The introduction text gives us the following notices:

> NOTE: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

And...

>NOTE 2: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

Duly noted.

Awesome we get to write our own scripts now! So let's have a look and see exactly what we have to do here. Log into the machine using the password from the last level, and lets take a look at the script in the `cron.d` folder.

```bash
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
    echo "Handling $i"
    timeout -s 9 60 ./$i
    rm -f ./$i
    fi
done
```

Huh, so it looks like this thing is moving to a directory called `bandit24` then executing everything in that folder... OK, so lets see if we can get something into that folder first, before we start with anything else.

Head over to `/var/spool` and run an `ls -l` command, let's take a look at those file permissions.

`drwx-wx--- 3 bandit24 bandit23 4096 Feb 27 15:47 bandit24`

OK, that's, not what you'd normally see... It seems we have write access to the folder, but we can't read anything in there (meaning we'll have to use a `/tmp` directory)

Well, write access is good, and we can just write the script locally and copy pasta it into the terminal, so we can deal with the "deletion" part pretty easily. OK, let's get scripting!

```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/bandit24pass
```

OK, so it's not exactly a Kernel module... But hey, it works!

NEXT!

## Level 24

> A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.
{: .prompt-info }

OK, before we get into this, we'll just try connecting to the daemon, giving it the password, and see what it fires back at us.

```bash
bandit24@bandit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
<Password for level24>
Fail! You did not supply enough data. Try again.
```

OK, that was expected. So, how are we going to go about bruteforcing this thing?

Well, there are a bunch of different ways, but I'm going to try and be a bit clever about this. We're going to use a wordlist.

To build our wordlist we're going to use a tool called `crunch`. `crunch` is an awesome tool that allows us to generate random wordlists based on a `min`, `max` and character set, for our purposes we'll use the following command:

`crunch 4 4 0123456789 > numberlist`

That will create a list of "words" that are a minimum of 4 characters, a maximum of 4 characters, and are made up of the numbers 0-9. Which gives us the following output, lettting us know that we have the right wordlist.

```bash
root@kali [15:01:12] [~/overthewire] 
-> # crunch 4 4 0123456789 > numberlist
Crunch will now generate the following amount of data: 50000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 
```

OK, so we have our wordlist, now we need to figure out how we're going to do this... I don't know about you, but I feel a Python script coming on!

(Quickly checks that Python is available on the target system, because we haven't actually had to use it yet.. YES, awesome)

Let's plan this Python script out then, otherwise we're going to be in a bit of a pickle (no pun intended). For a kick off we're going to need to read in our wordlist and then output it line by line into the daemon. So let's do that first bit first:

```python
import sys

with open numberlist as wordlist:
    for word in wordlist:
        sys.stdout.write(word)
```

Now we need to append the password to the current level in front of the `word`

```python
import sys

level24 = "PasswordGoesHere"

with open numberlist as wordlist:
    for word in wordlist:
        sys.stdout.write(level24+" "+word)
```

OK, so that's the output sorted, now we need to sort out connecting to the target. My initial idea was to use the `Requests` library, but there appears to be something wrong with their SSL installation for that particular module, so we'll have to resort to good ol' fashioned `socket`.

An issue arises almost straight away here:

```python
import socket

host = "localhost"
port = 30002

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((host, port))
sock.send("PassWordGoesHere"+"6969")
data = sock.recv(4096)
print data
```

We're being prompted for input _after_ we connect, not _upon_ connection. That makes this a slightly trickier prospect than I had originally envisioned. Now we're going to have to work out how to send the data _after_ we've established a connection.

With a binary, this could be achieved quite easily using `gdb` and `peda` to connect to the process and step through it. But in this case we're dealing with a websocket, so we're going to have to write something that can communicate with it as if we were typing into the terminal.

So, after sitting and thinking about this for a little while, I decided that I should probably leave Python out of it, we've not had to use much (if any) Python during this whole thing, and they aren't just going to throw something which is arguably pretty complex in a language which (as far as they know) we've had no exposure to. So let's go back to `bash`, since we have been using that in this wargame, and it's easier to use in this context than Python.

OK then, how do we do this in `bash`?

Well, luckily, since we're using `bash` we can just use the `|` (_pipe_) operator to send input around. So the first part of our script is pretty simple.

`echo $password $number | nc localhost 30002`

We can give that a try right away, to the BashMobile:

```bash
pass="PasswordGoesHere"
number="6969"

echo $password $number | nc localhost 30002
```

As you can see, we got the passcode wrong (and if you just copied and pasted you also got the password wrong as well), but that's expected behaviour. We know that 99999 of the passwords we're going to try are going to be incorrect, so we'll add in a check so that we can basically ignore that and keep moving. To do that we can simply change our command in `bash` to the following:

`echo $password $number | nc localhost 30002 | grep Wrong >/dev/null`

That last part will just look for the error message and dump it into the blackhole where we don't have to care about it anymore.

OK, cool so we can talk to the process, and we can get a response back. So let's turn this thing into a loop!

```bash
#!/bin/bash

pass="PasswordGoesHere"
FROMHERE=9999
for ((i=FROMHERE; i>=1; i--))
do {
    if
        echo $pass $i| nc localhost 30002 | grep Wrong >/dev/null
    then
        echo $i
    else
        echo $pass $i | nc localhost 30002
        exit
    fi
}
done
```

The astute amongst you will note that I decided to run the counter backwards. That's because I feel like i know how the minds of these CTF creators works, and they'll probably put it near the end somewhere.

To do the search from the bottom you'd do something like:

`for i in {1000..10000} ;`

Doing it this way also let me cope with the fact that the script kept stopping, which I'm pretty sure is related to my shoddy internet connection, but for now I'm going to blame my poor coding on crashing out. So now when it fails I can just adjust that `$FROMHERE` variable to whatever the last code I tried -1 is. Which is still an annoying way to do things, but it stops me going completely insane. (apparently it helps if I don't touch the terminal while it's running)

The one I have running while I type this has made it all the way down to the 9400's without crashing, so fingers crossed! (update: we've hit the 8900's!)

OK, so it finally completed, bloody thing kept crashing. And my initial guess that it was near the end was wrong, turns out these guys are butts and it was near the middle, so going from either end will take roughly the same amount of time.

## Level 25-26

So, the penultimate challenge. And it's an interesting one.

We're told to log in to Level 25 as normal, but then as soon as we get in (and run the obligatory `ls -la`) we can see that there is an ssh key, just laying around in the ~/home directory.

Naturally the first thing we're going to do is try to use it in the normal manner:

`ssh -i bandit26.sshkey bandit26@localhost -v`

(we add the `-v` flag because we're expecting something odd to happen with this connection)

And we're kicked off. As soon as we establish a connection our session is terminated and we're kicked back to Level 25's command prompt.

To get around this we've got to first figure out what the hell is going on.

`cat /etc/passwd | grep bandit26`

Outputs the following:

`bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext`

So it looks like we're using `showtext` instead of a normal shell, OK, cool, what the hell is `showtext`?

`cat /usr/bin/showtext`

```bash
#!/bin/sh

export TERM=linux

more ~/text.txt
exit 0
```

OK, so it looks like instead of spawning us a shell this `showtext` thing is using the `more` command on a text file then calling an `exit 0` (terminate).

Can we just read that file?

`cat /homebandit26/text.txt`

Nope :(

It also looks like there is a README.txt file in that same folder, which we should also see if we can read.

`ls -la /home/bandit26`

```bash
total 32
drwxr-xr-x  3 root     root     4096 Dec 28 14:34 .
drwxr-xr-x 29 root     root     4096 Dec 28 14:34 ..
-rw-r--r--  1 root     root      220 Sep  1  2015 .bash_logout
-rw-r--r--  1 root     root     3771 Sep  1  2015 .bashrc
-rw-r--r--  1 root     root      655 Jun 24  2016 .profile
drwxr-xr-x  2 root     root     4096 Dec 28 14:34 .ssh
-rw-------  1 bandit26 bandit26  430 Dec 28 14:34 README.txt
-rw-r-----  1 bandit26 bandit26  258 Dec 28 14:34 text.txt
```

_le sigh_ Well, we did note that it was using the `more` command, but when we run the application it doesn't appear to actually use it... We can change that behaviour quite easily, and in an odd way.

We're going to shrink our terminal down to a really small window size.

Doing this will force `more` to think that it needs to spool more text onto the page. Which gets us our _in_

`more` uses `vim`-like syntax. So, when we get the `more` command to fire off all we need to do is hit `v` and we're in `vim` mode.

From there we just need to poke around and see what we can use to break back into a proper terminal.

`:python print "Hello World"`

That gets us the following informative error message:

`E319: Sorry, the command is not available in this version`

OK, but what about Python3?

`:python3 print("Hello World")` (don't forget the braces)

Boosh!

That's it, we have command injection!

Now we just need to run the trusty Python command to spawn us a shell and we're golden.

`:python3 import pty; pty.spawn("/bin/bash")`

Bish, Bash, Bosh! We have a full shell, we could `cat` the password to the next level (if it existed), and we can now read that README file, which informs us of our obvious victory :)

```bash
Congratulations on solving the last level of this game!

At this moment, there are no more levels to play in this game. However, we are constantly working
on new levels and will most likely expand this game with more levels soon.
Keep an eye out for an announcement on our usual communication channels!
In the meantime, you could play some of our other wargames.

If you have an idea for an awesome new level, please let us know!
```

## Summary

I've really enjoyed doing this series of challenges, and I can't wait for them to add some more levels.

Join me next time, when I'll be having a crack at some of the other challenges on [OverTheWire](http://overthewire.org)
