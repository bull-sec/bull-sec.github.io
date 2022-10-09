# Inception HTB - 10.10.10.67

> This one was quite the journey, got our heads around a bunch of weird pivoting and got more experience with paying attention to execution context!

## Process Outline

- run nmap, find port 80 and 3128 (SquidProxy)
- fuzz the site on port 80
- find `dompdf`
[exploit-db 33004](https://www.exploit-db.com/exploits/33004)
- get arbitrary file read (not an LFI!!!)
- enumerate file system slowly (get username `cobb` from `/etc/passwd`)
- grab the apache config (000-default.conf)
- find webdav open...? 
- need creds, but the apache config file has the location
- pull them down, crack it with `john`

`webdav_tester:babygurl69`

- try a bunch of reverse shells, don't get execution (not sure why (correction, our code was bad)) end up using `phpbash`

[phpbash link](https://github.com/Arrexel/phpbash)

- get a better means of enumerating file system
- find wordpress_4.8.3 installation folder (not sure how or why you'd try this without the reverse shell)
- (made a list of ALL wordpress versions *255*, finds it instantly, see an error page, but at least we know it exists)
- pull down wp_config.php

```bash
curl http://10.10.10.67/dompdf/dompdf.php?input_file=php://filter/read=convert.base64-encode/resource=/var/www/html/wordpress_4.8.3/wp-config.php
```
> Could also just use `phpbash` since we went to the trouble of uploading it...

- decode the contents, find password for "mysql"
- password reuse is a major issue in our time
- no ssh...
- proxychains ssh cobb@127.0.0.1 **first layer**
- get user.txt

`4a8bc2d686d093f3f8ad1b37b191303c`

- start enumeration for priv esc
- `group`
- user `cobb` is in the `sudo` group
- sudo su
- we're done, right?
- nope
- run netstat -antp
- see weird ip
- machine IP is 192.168.0.10 (what?)
- 192.168.0.10 is talking to 192.168.0.1 over TFTP
...
- OK... here we go...
- tftp is writing as root, we get a success or fail, but we can't read files
- ftp CAN read files, but not write...
- idea.jpg
- use ftp to look for files
- find cronjob to update APT every 5 minutes
- find way to exploit for command injection

[APT Pre-install Hook](https://www.cyberciti.biz/faq/debian-ubuntu-linux-hook-a-script-command-to-apt-get-upgrade-command/)

- upload 00pwned and a reverse shell via TFTP
- first attempt fails... 
- no execute permissions on uploaded file??
- back to link, see that you can stack the commands
- modify file so first command sets the exec bit `+x` and second runs it
- launch nc on 192.168.0.10
- fiveminuteslater.jpg
- get root shell on 192.168.0.1
- get root flag! 

`8d1e2e91de427a6fc1a9dc309d563359`




