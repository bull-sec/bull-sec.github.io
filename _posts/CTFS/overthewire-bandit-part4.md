---
layout: post
title: "CTF: Bandit - Part 1"
date: 2022-07-30 23:32 +0100
tags: pentesting linux kali overthewire
categories: [CTF, OverTheWire]
published: false
---

## Overthewire - Bandit Part Four(?)

Right... so erm... it turns out they updated Bandit.

Back to it then!

## Getting Back in the Saddle

When we last left off (back in December of 2017) we had just completed challenges 25 and 26, and upon completion of those challenges we were given a simple README file congratulating us for completing the wargame and asking for content submissions.

Guess they got some.

That does mean we're going to have to go back to Level 26 to get a password for the next level, which is a tad annoying but whatever we'll crack on.

## Level 26 (Redux)

OK, won't daudle on this one because I've already covered it in (what was supposed to be) the last post.

... Or maybe I will... Because as Dr. Dre said, "Things just ain't the same for gangsters", and that holds true for us in our current situation. The nice and simple (once it's been explained) breakout doesn't appear to work anymore, so let's go digging and find out why!

Well that was lame, the original way that we used in the last post was so much cooler...

Instead of using python to spawn a shell, we just need to use vim's "Explore" function which is accessed by either `:E` or `:Explore`.

Navigate to `/etc/bandit_pass/bandit26` and hit enter, get the password, feel bad about yourself for coming up with something dope that doesn't work anymore.

Oh well.

Except we're not quite done...

Let's use that password to log back into `bandit26` as normal. Now we'll deal with the shell a different way.

Same thing again, shrink the terminal down so `more` fires, but this time instead of using python to spawn a shell we're just going to use `vim`'s awesome functionality.

Go into `vim` mode in `more` and enter the following:

`set shell=/bin/bash`

Then hit enter and type:

`shell`

And you should be dropped into a fully responsive and non-funky bash shell.

If you `ls` you'll see some interesting files:

```bash
bandit26@bandit:~$ ls -la
total 36
drwxr-xr-x  3 root     root     4096 Oct 16 14:00 .
drwxr-xr-x 41 root     root     4096 Oct 16 14:00 ..
-rwsr-x---  1 bandit27 bandit26 7296 Oct 16 14:00 bandit27-do
-rw-r--r--  1 root     root      220 May 15  2017 .bash_logout
-rw-r--r--  1 root     root     3526 May 15  2017 .bashrc
-rw-r--r--  1 root     root      675 May 15  2017 .profile
drwxr-xr-x  2 root     root     4096 Oct 16 14:00 .ssh
-rw-r-----  1 bandit26 bandit26  258 Oct 16 14:00 text.txt
```

Running `file` on that `bandit27-do` shows us that it's a 32bit ELF, and the `ls` command showed us that it has the sticky bit (SUID) set in it's permissions (also it's bright freaking red!) Now if you were to just try to run that file, like so:

`./bandit27-do`

You'd be greeted with the following message:

```bash
bandit26@bandit:~$ ./bandit27-do
Run a command as another user.
  Example: ./bandit27-do id
```

Huh, just "a command", what, so `cat` will work?

`./bandit27-do cat /etc/bandit_pass/bandit27`

Yep!

We're back baby!

## Level 27 (New Ground!)

*sniffs* Ahh the smell of a fresh challenge. And in the spirit of making things clearer I'll be posting the challenge description at the start for the rest of the challenges.

> There is a git repository at ssh://bandit27-git@localhost/home/bandit27-git/repo. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.

Cool, let's do this!

So upon loggin in we'll do a quick `ls` see that there is nothing of note in the folder, and head off to a `/tmp` directory where we know we'll have full read/write access regardless of shell restrictions (also it's cleaner working in `/tmp` \#OpSec)

Sadly this challenge wasn't exactly challenging (or I've just gotten a hell of a lot more competent and I've not realised)

`git clone ssh://bandit27-git@localhost/home/bandit27-git/repo`

Now `cd` into the directory and read the README.

Done. Moving on.

## Level 28

Another git repo... Hmmm

Aww, that was lame (OK I'll admit at this point I've learned a hell of a lot since I first did this so these challenges really are feeling rather simple)

Same as before, clone the repo to a `/tmp` directory and `cd` into it. Now if you check out `README.md` the password has been obfuscated. However there is a nice simple way around this minor inconvinience.

`git show`

That'll drop out the diffs for the last commits (if you have no idea what I'm talking about go research Git, NOW!) and in them we can see a nice shiny password.

## Level 29

More of the same, only this time we're introduced to the concept of branches.

The README states:

> No passwords in production!

OK, so that means that they ARE allowed in non-production environments, right?

`git branch`

Will show us the branches that are available.

`git show origin/dev`

And there again we see a password that was "redacted" but remained in the git commits.

The lesson to learn here is thusly; GIT NEVER FORGETS! (unless you revert the HEAD but who does that in the real world?)

## Level 30

(You may have noticed that the promised descriptions aren't making themselves known. That's because they're all the same so far and I didn't want to waste precious pixels repeating nearly the exact same information, the only changes between them are the host names)

OK, this one is proving to be a bit tricky and I retract my previous statements about this being too easy!

Faced with the same challenge as last time (x2) except this time we have nothing but an initial commit, no branches, and not a lot to go on.

Digging around in the `git` commands we don't get much help from `show` or `diff`, as there is nothing to show and therefore nothing to diff. Sad times. But all is not lost.

`cat`'ing through the .git directory there are a few text files that we can read, the most interesting of which is `packed-refs`:

```bash
bandit30@bandit:/tmp/balls/repo/.git$ cat packed-refs 
# pack-refs with: peeled fully-peeled 
3aa4c239f729b07deb99a52f125893e162daac9e refs/remotes/origin/master
f17132340e8ee6c159e0a4a6bc6f80e1da3b1aea refs/tags/secret
```

Secret, ay?

But how the heck do we access it?

Googling around the answer is "We don't" or at least not easily, however it did point us in another direction that we should investigate:

`ls -laR | grep 'idx\|pack'`

That'll show us the location of the `pack` and `idx` files (duh)

[Git Packfiles Documentation](https://git-scm.com/book/en/v2/Git-Internals-Packfiles)

Seems we can use `git verify-pack` to take a look at the `pack` file and see what's inside:

Although that didn't seem to do anything... Hmmm

So at this point I resorted to the manual:

`git <command> --help`

Until I found `git cherry`

I tried `git cherry secret` and got the following result:

```bash
bandit30@bandit:/tmp/balls/repo$ git cherry secret
error: object f17132340e8ee6c159e0a4a6bc6f80e1da3b1aea is a blob, not a commit
fatal: Unknown commit secret
```

Blobs are objects that are yet to be commited or indexed (as far as I can tell, read the documentation don't take my word for it) so that leads us to the following documentation:

[Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)

Where we find the following command:

`git cat-file`

That's our ticket to freedom.

`git cat-file -p f17132340e8ee6c159e0a4a6bc6f80e1da3b1aea`

Where `-p` just means to pretty-print the output, and that long value is the name of the object that we got when we tried to open it with `cherry`.

Fire that off, and bask in your passwordy glory!

## Level 31

Please no more git, please no more...

Dammit...

OK, first we check out the README.md and see the following:

```bash
bandit31@bandit:/tmp/gorilla01/repo$ cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master

```

OK, so we have to craft a text file and upload it, simple right?

```bash
echo "May I come in?" > key.txt
git add .
git commit -m "May I come in?"
git push -u origin master
```

Except, no...

```bash
bandit31@bandit:/tmp/gorilla01/repo$ git push -u origin master                                                                                                                                                  
Could not create directory '/home/bandit31/.ssh'.                                                                                                                                                               
The authenticity of host 'localhost (127.0.0.1)' can't be established.                                                                                                                                          
ECDSA key fingerprint is SHA256:98UL0ZWr85496EtCRkKlo20X3OPnyPSB5tB5RPbhczc.                                                                                                                                    
Are you sure you want to continue connecting (yes/no)? yes                                                                                                                                                      
Failed to add the host to the list of known hosts (/home/bandit31/.ssh/known_hosts).                                                                                                                            
This is a OverTheWire game server. More information on http://www.overthewire.org/wargames                                                                                                                      
                                                                                                                                                                                                                
bandit31-git@localhost's password:                                                                                                                                                                              
Branch master set up to track remote branch master from origin.                                                                                                                                                 
Everything up-to-date  
```

Ummm, OK... well let's check out the `.gitignore` and see why that happened, although at this point I have a good idea.

```bash
bandit31@bandit:/tmp/gorilla01/repo$ cat .gitignore 
*.txt
```

Of course... But we are not defeated, luckily we are able to use the `-f` or `--force` command to push files up that are on our `.gitignore` file. Yay!

All we need to change from our initial command is the following:

```bash
git add -f key.txt
```

Then `commit` and `push` and you'll be greeted with and error, fret not though because it contains the password to the next level!

## Level 32

Oh, yay! We're done with `git` apparently. And this is also the last level (for now)

> After all this git stuff its time for another escape. Good luck!

"Good luck", that doesn't bode well...

OH DEAR GOD!

Upon logging in we're greeted with the following message...

```bash
WELCOME TO THE UPPERCASE SHELL
```

And indeed upon entering a simple `ls` we get this wonderful message:

```bash
WELCOME TO THE UPPERCASE SHELL
>> ls
sh: 1: LS: not found
```

OK, all is not lost... well actually it kind of is (I'm typing this while giggling) it's mostly impossible to do anything. I managed to do `$path` which got converted to `$PATH`... so that was *some* progress.

![meh](images/meh.jpg)

That took a while, I probably would have gotten there eventually but in the end I cheated. I still don't really know how or why what I did worked, but it did (and the guy I stole it from [This Guy!](https://shawnduong.github.io/overthewire-bandit-writeups/) didn't seem to know why either)

```bash
$0
```

![spongbob](images/spongebob.gif)

Til we meet again!
