# Healthcare - Vulnhub

Healthcare is an "Intermediate" box made by @v1n1v131r4 that aims to teach the student the OSCP methodology, aka, Try Harder. 

## Recon

First we run a full port scan using `nmap`:

```bash
nmap -p- -Pn -vvv -n -oN ./nmap_results/all_ports_192.168.56.109 192.168.56.109
```

*Results*

```bash
Nmap scan report for 192.168.56.109
Host is up, received user-set (0.00039s latency).
Scanned at 2020-09-08 13:10:14 EDT for 4s
Not shown: 65533 closed ports
Reason: 65533 conn-refused
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
80/tcp open  http    syn-ack
```

If we take those ports and do a Service Verification on them we get a bit more information:

```bash
PORT   STATE SERVICE REASON  VERSION
21/tcp open  ftp     syn-ack ProFTPD 1.3.3d
80/tcp open  http    syn-ack Apache httpd 2.2.17 ((PCLinuxOS 2011/PREFORK-1pclos2011))
|_http-favicon: Unknown favicon MD5: 7D4140C76BF7648531683BFA4F7F8C22
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 8 disallowed entries 
| /manual/ /manual-2.2/ /addon-modules/ /doc/ /images/ 
|_/all_our_e-mail_addresses /admin/ /
|_http-server-header: Apache/2.2.17 (PCLinuxOS 2011/PREFORK-1pclos2011)
|_http-title: Coming Soon 2
Service Info: OS: Unix
```

## FTP - Port 21

- Tried `anonymous` as a user, wasn't able to access


## HTTP - Port 80


## Priv Esc




## Flags

```bash
d41d8cd98f00b204e9800998ecf8427e:user.txt
```


```php
cat /root/root.txt
██    ██  ██████  ██    ██     ████████ ██████  ██ ███████ ██████      ██   ██  █████  ██████  ██████  ███████ ██████  ██ 
 ██  ██  ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██ ██ 
  ████   ██    ██ ██    ██        ██    ██████  ██ █████   ██   ██     ███████ ███████ ██████  ██   ██ █████   ██████  ██ 
   ██    ██    ██ ██    ██        ██    ██   ██ ██ ██      ██   ██     ██   ██ ██   ██ ██   ██ ██   ██ ██      ██   ██    
   ██     ██████   ██████         ██    ██   ██ ██ ███████ ██████      ██   ██ ██   ██ ██   ██ ██████  ███████ ██   ██ ██ 
                                                                                                                          
                                                                                                                          
Thanks for Playing!

Follow me at: http://v1n1v131r4.com


root hash: eaff25eaa9ffc8b62e3dfebf70e83a7b
 
```


## Process

- Enumerate Target
- Smash head against fake Shellshock exploit
- Eventually find correct wordlist
- Find openemr (medical records database)
- FInd SQL injection in 'validateUsers'
- Dump admin password
- Use photo upload to get reverse shell
- find backup copy of shadow file
- pivot into user shell *not actually required*
- Run `linpeas.sh`, nothing interesting pops up
- Find SUID binary called `healthcheck` (non-standard binary)
- Run `strings` on it, see that it's using relative paths
- Modify `$PATH` to point to current directory
- Move to writeable directory (/tmp/)
- `echo "/bin/bash" > ifconfig`
- Run binary, get root shell 