---
layout: post
title: "CTF: Bandit - Part 1"
date: 2022-07-30 23:32 +0100
tags: pentesting linux kali overthewire
categories: [CTF, OverTheWire]
published: false
---

## OverTheWire - Bandit - Levels 0-9

[Bandit Wargame](http://overthewire.org/wargames/bandit/)

The following is a short and snappy walkthrough (post edit: LOL it's really not, it goes on forever) for the Bandit wargame on OverTheWire dot org. I wrote it mostly as an exercise in .MD formatting, and blog posting, but it should be helpful for those who are stuck on particular sections of this rather fun little wargame.

Oh, a quick note, as some people who are reading this may be new to Linux and POSIX systems in general. There are some basic commands you will need to know before you can actually do any of this. And they are as follows.

```bash
ssh     - ssh <user>@<server>, if you aren't using Putty
find    - runs the aptly named find program, it finds things
ls      - lists the files in the current directory
grep    - regular expression parser, helps filter data from stdout
```

Most Linux programs have a built in help function that will display some of the basic commands. This is usually accessed by typing the name of the program and then `-h` or `--help` in some cases.  

`man`

I left that one on its own because its important. In fact `man` may very well be one of the most important commands you ever learn on Linux. `man` = Manual. And most programs have one, and they generally have a lot more detailed information than the help menus. Or in some cases they are just carbon copies, either way, learn to use the help and `man` pages.

I should also note that there are probably (definitely) multiple ways to solve these challenges, these are just the methods I used. Have fun! :)

## Bandit 0

Super simple, first boot up putty (or your SSH client of choice) and log in using the given username and password

username = `bandit0`
password = `bandit0`

Then run an `ls` command to see where you are in the world (file structure).

You should see the file `readme` in your current directory. Readme files should always be read, it is in fact why they are called readme files.

`cat readme`

And BOOM! You should have your first password printed on the screen in front of you. It won't be a word it'll be a base64 encoded string

Copy pasta that into a notepad (I saved the commands as well as a short description of the challenge, just for reference)

Disco from the server (you're done here) and boot your SSH client back up ready for the next challenge (You'll be doing this after every challenge)

## Bandit 1

Now we start having to do some stuff!

This first challenge contains a file called `-`

Linux treats the `-` character as a special character, and trying to open the file using the previous method (i.e. `cat readme`) isn't going to work. This isn't too tricky to get around, you just need to force `cat` to treat your special character as a filename instead of a command. To do this you just need to give cat an absolute filepath.

`cat /home/bandit1/-`

Bish bash bosh. Moving on!

## Bandit 2

This challenge thinks it's being pretty slick, but we can get around the issue of having spacess in the filename quite easily. Using our old friend `cat` we can pass the filename in as a string by using the `'` character.

`cat 'spaces in this filename'`

NEXT!

## Bandit 3

This one could catch you out if you aren't that up on your bash commands.

But luckily, tab-completion has our back (i.e. press tab and it'll auto-complete the command for you). To get the password for this machine simply type `cat` then the directory name `inhere/` and hit **TAB**, the console should autocomplete the rest for you.

But this is cheating (a bit)... So we'll run the proper command first.

Type `ls -la inhere/` and it should show you all of the files in that directory, hidden or not. The `-la` flag tells `ls` to list all of the files, regardless of their hidden status.

You should see a sneaky little file in that directory, that wouldn't otherwise be visible using a standard `ls` command. Something to bear in mind when using a terminal(remember, if you don't know how to use something, there is no shame in consulting the `man` page or using `-h`/`--help`)

OK, by some combination of the above methods you should have the password for the next machine. So lets close this box down, and get ourselves logged in for the next challenge!

## Bandit 4

This one is actually a pain in the arse, and I've not found a single command that'll do it yet.

I think this one is designed to teach you how to use the `clear` command more than anything (although I may just be an bad)

Run our trusty `ls` command to see the contents of the `inhere` directory, and behold the bounty that lays within! OK, so its a few files, and most of them aren't readable, but a bounty is a bounty!

I'm really unhappy with this one, but I basically just ran `cat <filename>` on each one until it didn't break the terminal (you'll see what I mean)

`cat inhere/-file01`
`clear`

Repeat until password :(

LATE BREAKING EDIT!

As I said, I was really unhappy with this one, so I went back and had another crack "this has to be possible in a single command" I thought to myself. And low and behold, it was!

`cat inhere/-file0* | strings -e S`

That'll poop out a load of garbagio, but in the middle of the garbage should be a rather obvious looking string that isn't garbled in any way, and doesn't break the console, and that is your password! You're welcome! Ish, its still terrible and I can probably clean that input somehow...

NEXT!

## Bandit 5

This one threw me at first, because I'd never used the `find` command beyond basic string searches. But it turns out that you can find more than just filenames! Who knew?

`find inhere/maybehere* -size 1033c`

The `c` is SUPER IMPORTANT, and I got very frustrated until I read the `man find` and realised that the default search was for bits, and bytes had to be specified.

That command should spew out the right filename, then just run our old friend `cat` to get the new password and we'll move on!

## Bandit 6

If you actually read the `find` man pages then this one is fairly straightforwards. `find` has two methods which will be of great use here, `-group` and `-user`. We are helpfully given the file size as well, so this should help to focus our search a little bit.  

`find / -group bandit6 -user bandit7 -size 33c`

DAMN THAT'S A LOT OF OUTPUT!

OK, so it's fairly obvious that this VM has limited permissions at this point... But wait, what's that at the top of the list?

(Note: a nice little trick I've found for the `find` command, is to pipe any error messages into `/dev/null`, it just makes life a bit easier, you can do this by adding `2>/dev/null` to the end of the string, and the only thing that should return (in this case) will be the file we're looking for)

Got it? Awesome, lets keep rolling!

## Bandit 7

Simple bit of grepping here, nothing too fancy.

`grep 'millionth' data.txt`

That should spool out a familiar looking string. Grab it, paste it into your text file and lets crack on!

## Bandit 8

Bit of sorting to do in this challenge. The `data.txt` file contains a load of values, a lot of which could potentially be our password. If you run a basic `cat data.txt` you'll see them all pop up.

To sort these files we will use the aptly named `sort` command.

`cat data.txt | sort`

But that didn't really do much, just put things in a nice ordered list for us, which I suppose is nice, but its not what we want. So we need to find the unique values in this list, using another aptly named command `uniq` with a `-u` flag, which simply tells `uniq` to only print the files which occur only once.

`cat data.txt | sort | uniq -u`

It's important to remember that the `sort` command goes before the `uniq` command. It's a bit of a quirk, but you only have to remember it once.

Oh right, password... So you have it now? Yes? Great stuff, onto the next challenge!

## Bandit 9

OK, so I figured most of this one out by myself, but I ran into an issue I had to Google, that being null data.

This challenge has you looking for a human-readable string in among a bunch of garbagio. The only thing we have to go on is that the string we are looking for starts with "several" `=` characters (which incidentally sparked off a debate among me and my friends about what the definition of "several" was)

The initial grep string I used was as follows:

`cat data.txt | grep -F '==='`

But this just spat out a message stating that the "Binary file matched" or some such notification. This would not stand, so I Googled the message and came across a stackoverflow post talking about filtering null data from a file using the `tr` command. Perfect.

`cat data.txt | tr -d '\000' | grep -F '==='`

The output is still a bit garbled, but you should see the telltale string that is the next password with "several" `=` characters highlighted by `grep`

[Find Part 2 Here](/posts/bandit-part2)
