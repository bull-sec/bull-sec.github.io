# Leviathan

> Dare you face the lord of the oceans?

Yes, I very much dare.

## Introduction

Similar to the last challenge (Bandit) we have to log into the machine via SSH. 

`leviathan.labs.overthewire.org`

Note: we also need to use `port 2223`

The user credentials are in the following format:

```
Username: leviathan0
Password: leviathan0
```

We're also told that we can find all of the passwords in the respective `/etc/leviathan_pass/` folder for the user we're trying to crack.

Other than this, I have no idea what to expect in terms of challenges and difficulty level. So let's just dive in and figure it out as we go!

## Level 00

Log in. If you can't handle that then I don't know what to say to you...

## Level 01

Running our obligatory `ls -la` command reveals a "hidden" folder called `.backup`, which has a html file called `bookmarks` in it. 

I had no real idea what to do with this one at the start, so I copied the HTML file onto my local machine and opened it in a browser. Took me about a minute to spot a rather suspect link, which was something like "the password for the next level is..."

We can make this a bit easier on ourselves:

`grep password bookmark.html`

And that'll reveal it's secrets.

## Level 02



