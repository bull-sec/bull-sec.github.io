#OverTheWire - Bandit - Levels 10-19
###Wargame Walkthrough by HybridGorilla


##Bandit 10
This is another one that thought it was slick. Nothing much to this at all, especially if you've been paying attention to the "Commands you may need to solve this level" hints that the developers kindly give you. 

We're going to use the `base64` command to decode the text file and poop out our coveted password.

`cat data.txt | base64 -d`

Note: `base64` only has a few commands, so it's worth committing to memory as it comes in handy.

OK, that wasn't too painful was it? Lets move on, to...

##Bandit 11
OK, so I got the gist of this one right away, but I still had to Google the answer, and even then I messed up the command line argument about 10 times. The challenge has you decode a Caesar cipher, and if you don't know what one of those is then you need to stop doing this and go find out, [Wikipedia is usually a good starting point for a broad view (but don't rely on it)](https://en.wikipedia.org/wiki/Caesar_cipher)

The hard part of the decryption has been done for us, as we have been given the key, or the number of characters the alphabet has been shifted by. Normally we'd either have to brute force it (which wouldn't take too long with even a long Caesar Cipher, they aren't the strongest) or perform some frequency analysis on the letters in the string to determine the correct shift key.

```
abcdefghijklmnopqrstuvwxyz
            ^
           13
```
So, we know that the text has been shifted by 13, and by working out what the 13th character in the alphabet is (its `m` if I hadn't made it clear enough) we can work out how to rotate our string so that the garbled text makes sense (I'm assuming at this point you went ahead and did a `cat` on the file just to see the output)

`cat data.txt | tr a-zA-Z n-za-mN-ZA-M`

Let's break that command down because there is a fair amount going on...

`cat data.txt` 
You should be more than familiar with this by now.
`|`
 Our trusty pipe command, which moves the output from cat into the command we specify next.
 
Right, now the complicated bit:
`tr` Our initial command which calls the translate or transliterate program, which isn't all that powerful but its pretty useful as it turns out.
`a-zA-Z` 
Tells tr that we want to apply the translation to all uppercase and lowercase letters.
`n-za-mN-ZA-M` 
This is where our magic is happening. What we are doing here is telling `tr` that it should replace `m` with `z` and `a` with `n` (both in lower and uppercase forms).  If it looks a bit funky, its because it is, and I'm not entirely sure why.

That means our new alphabet reads as follows:

nlopqrstuvwxyzabcdefghijklm

Now if you run that whole command from earlier you should get a nice easy to read string that says "The password is..." And we can move on :)

##Bandit 12
Things are starting to get tricky now, here we have hashdump of a compressed file, and it's also the first time you'll have to work with a temporary directory, as you don't have permission to modify or create files otherwise. 

So, first things first, lets create that temporary directory.

`mkdir /tmp/<dirname>`

Call the directory whatever you like, I called mine `mist` because I was feeling a bit mysterious vOv

We're given two clues right off the bat, first is in the suggested commands `xxd` which is an awesome little program that performs witchcraft!

`xxd -r data.txt /tmp/mist/bandit`

This will create our 'reverted' file which we will use for the rest of this challenge. The thing to note in this command, and the bit I forgot to copy over for some reason, so when I came back to this to check it before posting it didn't make any sense, is the `-r` flag. Which, from the `xxd` help page, reads:

`-r          reverse operation: convert (or patch) hexdump into binary.`

That's what we want to do, create an actual binary file from a hexdump, all we were doing without the `-r` flag was hexdumping a hexdump, and that wasn't going to get us anywhere fast!

For the sake of brevity I called the newly created file `bandit` in each case.

`file bandit`
`mv bandit bandit.(compression function) - as indicated in output from file`
`(decompression function) bandit.(whatever)`

So what are we doing here? 

We are going to check the file type using `file`. Then we will `mv` or move our 'reverted' file, essentially renaming it with the correct extension as indicated in the output from `file`. From there we will simply apply the correct decompression function to the newly created file and then follow the process until the `file` output states that the file contains text instead of being a compressed binary. 

If you've made it this far you should be able to handle it without much problem. Just remember, if you get stuck `man <program name>` your way to success, you're given a hint on each level as to the commands you should be using, so if you're struggling, just `man` each of those and the answer should spring out at you. And if that fails, there is always Google. 

Got that password? Good stuff, we're about halfway there, and if you've been ploughing through this then now might be a good time to take a little break, because the rules of engagement change a little bit for the next round of challenges. 

##Bandit 13
At this point in the proceedings we are having to do some slightly more complex functions, and we'll also be introduced to a bunch of new commands and programs. 

Bandit13 asks us to retrieve a password from a file which can only be read by a specific user. Luckily though, someone has gone and left a private key laying around.

Log in to Bandit13 and run the following command:

```
ssh bandit14@localhost -i sshkey.private
```
You'll be asked if you want to accept the ECDSA fingerprint, say yes, and you should get logged in as bandit14.

All we need to do now is run our trusty `cat` command on the filepath we are given in the specification and we should get our shiny password for the next level. 

##Bandit 14
OK, log in to Bandit 14 using the password we just captured in the previous challenge.

Now its time to start playing with netcat! For those who have never used it before, netcat is like a swiss army knife for networking, super useful program that I highly recommend people learn how to use. 

Netcat is fairly simple to use, at least for what we need to do here, which is to send a message to a remote device and receive a response. Enter the following command into a terminal:

`nc localhost 30000`

That will create a connection between you and the server. Now if you type into the command line you are actually talking to the server. So lets do what it wants and paste in our **current** password (i.e. the password for level 14)

If you've entered it correctly the server should respond and confirm this by giving up the goods, and printing out the password for the next level.

##Bandit 15
Very similar challenge to the previous, only this time we are being asked to connect to an SSL port.

Run the command `nc --help` and you'll note that our beloved netcat does not actually have dedicated function for connecting to an encrypted port or service, which sucks. But fret not, because the developers have given us a big ol' clue as to how to get around this particular problem in the "Commands you may need" section. 

Note: the developers also give you a hint on what to do if you get an error message ()

So, looking at the list of suggested programs we can strike a bunch off right away. SSH isn't going to work, we want SSL, the letters are different for a kick off. Telnet isn't going to be much use either, its not designed to handle encrypted traffic (that's why we have SSH and SSL!) We've already covered netcat, and Nmap isn't going to be much use to actually connect to a machine (we'll cover Nmap more in the next challenge). So that leaves us with `openssl`. Which is fine, because its the exact tool we need for the job!

In your command line, enter the following (note: I needed to use the workaround provided by the developers, you may not have to, but I've provided it anyway)

`openssl s_client -connect localhost:30001 -ign_eof`

Once that connection is established, simply copy pasta your current password into the terminal and hit enter to get the password for the next level. Bish bash bosh.

##Bandit 16
Nmap! Now we're talking! 

I love Nmap, it is probably my favourite networking tool. It's another one like netcat which I strongly recommend learning the flags for, it is so useful when conducting penetration tests, or even just doing basic networking things. Small example, a few weeks ago the DHCP lease on my smart-switch ran out and I couldn't find it in the ARP tables on my router, so I used Nmap to scan the network for machines running services on that port. Simples.

We're going to be doing something similar during this challenge. We've been given a range of ports that a service might exist on (31000-32000) and we're going to need to perform a quick scan of that range to locate our desired connection. 

Running a basic `nmap localhost` command won't get us very far here, as we need to scan a specific range.

`nmap --help | grep "range"`

Get used to running commands like the one above, it'll save you so much time. 

`nmap localhost -p "31000-32000" -Pn`

Hmm, that hasn't quite got it has it? I mean we have some services we can connect to, but we don't have much information about them, and connecting to each one in turn sounds like hard work... Lets remedy the situation then.

`nmap localhost -p "31000-32000" -Pn -A`

Ahh that's more like it. Now, you should notice that most of the servers are running a service called `echo`, so those aren't the ones we're interested in, so we can strike them off the list right away (we're told this in the spec as well)

For extra clarity (and since we aren't bothered about leaving footprints on this machine) we can output the results of the `nmap` scan to a file using `-oG` then we can use the power of `grep` to get a much more concise list of available ports.

`nmap localhost -p "31000-32000" -Pn -A -oG /tmp/filename`

Right, now we just need to try the remaining services to find out which one is the gatekeeper. 

`openssl s_client -connect localhost:port`

Ooooh, what's this that has appeared? That isn't the same passwords we've been given previously... By George its an RSA Key!

Copy paste this key into a text document, you want to grab the entirety of it, including the header and footer. 

```
-----BEGIN RSA PRIVATE KEY-----
RANDOM GARBLED STUFF THAT IS THAT ACTUAL KEY GOES HERE
-----END RSA PRIVATE KEY-----
```

OK, so this next set of instructions is specifically for those who are using Putty to do these challenges. (Scroll down a bit further for the Linux setup instructions)

If you installed Putty from the .msi you should have also installed a program called PuttyGen. This is the tool we're going to need to create the key to get onto the next level. If not then you need to grab PuttyGen from the [official website](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

To convert the captured RSA key into the correct format we're going to need to save it as a `.ppk` file. Once we've done that, open up PuttyGen and load the newly saved key. You should get a nice little ping, and a message telling you that you need to save the private key before it can be used. It isn't lying, so follow its instructions, probably save it with a `new` prefix or something, just in case you mess up and need to revert to the original.

Now, bring Putty back up, but instead of simply connecting, we're going to go into the left hand menu. 
```
Under "Connections" click on "SSH".

Then click on "Auth"

Hit the browse button in the box labelled "Authentication parameters"

Browse to the location you saved the key we generated in PuttyGen and get it loaded up

Now hit the "Open" button on Putty

You will not be prompted for a password this time, just enter the username and you're good to go
```

OK, so that's the Windows lot sorted, now for my Linux homies, I didn't forget about you!

In 'Nix it's a bit simpler, just make a new file (call it something like privatekey.private), and copy the contents of the RSA key into it. Then in your terminal (while you're still logged in to bandit16) type the following:

`ssh -i privatekey.private bandit17@localhost`

After that you should be promted to perform the usual SSH mating rituals before being allowed to log in as bandit17.

Phew, all of that just so we could crack on with...

##Bandit 17
The hardest bit of this one is getting the RSA key sorted, there really isn't much going on with this particular challenge. 

We're asked to compare two files, and the password will be the only line in the `new` file which has changed. Simple enough. Lets introduce the tool for the job, `diff`. As its name suggests `diff` is used to find the differences between two files. And its rather good at it. 

Enter the following command into the Bandit 17 terminal: 

`diff password.old passwordnew`

I'll refrain from printing out the password, suffice to say that `>` = inserted, and `<` = removed :) 

If you're in doubt just try both.

Got it? Good stuff!

##Bandit 18
"The password for the next level is stored in a file readme in the homedirectory."

OK, simple enough.

"Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH."

What? Well shit, that puts a bit of a dampener on our day doesn't it?

We're also not really given much in the way of hints here, all we're given is `ls`, `cat`, and `ssh` as clues. So lets take one of those clues and try and do something with it. 

We can't connect to Bandit 18 directly through Putty, as the connection is just dropped instantly, so what if we take a step backwards and try to connect through our previous challenge? Boot up Bandit 17 again (you did keep the key, right?) and enter the following command into the terminal.

`ssh bandit18@localhost`

Accept all the warnings and such and enter the Bandit 18 password when prompted.

Annnnd, it didn't work... But at least we still have a terminal to type into. Silver linings and all that.

So the problem here is pretty simple, someone added an `exit` line into the **.bashrc** file and we don't even get a chance to enter anything into the console before we are unceremoniously ejected. Let's try and get rid of the **.bashrc** file then.

We can do this by just not using `bash` at all. Which is one of those things that makes so much sense it's baffling that I didn't think of it sooner. I was trying to get actually move files around on the target machine and do all kinds of crazy ish, but it turns out, the simple solution is the best.

`ssh -t bandit18@bandit.labs.overthewire.org -p 2220 /bin/sh`

And we're in! Happy days!

Run a quick `cat` and grab the password from the `readme` file, but don't log out yet, because there is some housekeeping to do. Execute the following command in the Bandit 18 terminal. 

`mv .bashrc-old .bashrc`

This will revert the system to its previous state. Now log out and just check that your connection is being dropped the same as it was before. There we go, tracks have been partially covered, password has been secured, and we learned how to get around a pesky little problem. Jobs a good'un!

NEXT!


##Bandit 19
Mmmmm SUID binaries....

After the last one I thought the only way was up, but this one is actually fairly simple (mostly because it's acting as an introduction to SUID binaries). We're being asked to perform a spot of privilege escalation using a binary file which has been provided for us.

Before we start, a quick introduction to SUID binaries. A SUID binary is a file which is created with the full permissions of the user that created it, meaning if a user with elevated permissions made the file (e.g. root), then it is running as that user, essentially giving us an inroad to elevate our own permissions on the system.

First thing you should do when you log in is run an `ls` to see where we are, as usual. Just the one file this time, and its the binary we were told about in the brief, good stuff.

Run the binary to see what its output is:

`./bandit20-do`

That will spit out the following message: 

```
Run a command as another user.
  Example: ./bandit20-do id

```

Well that is pretty self explanatory, and we've already been informed as to where the passwords are stored (/etc/bandit_pass). So lets just go for a `cat` and see what happens. 

`./bandit20-do cat /etc/bandit_pass/bandit20`

That would appear to be a password that just appeared on our screen! Excellent, lets get cracking with the next challenge!

Note: if you didn't also try to get some other passwords using this same method then you have brought shame to our dojo.
