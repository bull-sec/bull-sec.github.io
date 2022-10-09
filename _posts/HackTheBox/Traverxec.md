### System IP: 10.10.10.165
#### Service Enumeration

Protocol | Ports Open
------------------|----------------------------------------
**TCP** | 22,80
**UDP** | N/A

**Nmap Scan Results:**

```
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nostromo 1.9.6
```

**Vulnerability Explanation:**

Based on the Nmap report we noticed that the target was running a webserver called "nostromo". A quick search using Google brought up a number of vulnerabilities for the exact version which appeared to be running. 

![Google search results](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/nostromo_google_results.png)

The issue which all of the results seem to be referencing is CVE-2019-16278 which allows for remote command injection against the Nostromo web server. 

We downloaded the following public exploit:

[Nostromo RCE - CVE-2019-16278](https://www.exploit-db.com/exploits/47837)

![Command Execution via Public Exploit](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/command_execution_via_python_script.png)



**Vulnerability Fix:** Upgrade to a newer version of Nostromo.

**Severity:** High

**Local.txt Proof Screenshot**

![user shell](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/user_shell.png)

**Local.txt Contents**

7db0b48469606a42cec20750d9782f3d

#### Privilege Escalation

**Vulnerability Exploited:**

*www-data > david*

Once we had our reverse shell on the machine as the www-data user we did our usual enumeration steps to look for a way to escalate our privileges. Since the target was running the Nostromo webserver we checked out the configuration file and noted that there was a path that looked like it was pointing to a users home directory. 

A quick Google search told us that the Nostromo application allows users (and in our case service accounts) to read the contents of directories that are specified in the Nostromo configuration file. In our case that means we can read the contents of /home/david/public_www.

![David home folder contents](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/david_public_www.png)

Inside the readable folder "protected-file-area" is a pair of files:

![Protected File Area Contents](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/protected_file_area_contents.png)

Using John the Ripper we were able to extract a password from the .htaccess file, though it didn't appear to be used anywhere.

The contents of the `.tgz` file were extracted and found to be far more interesting as they contained a copy of both the id_rsa and id_rsa.pub for the `david` user:

![Tar file contents](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/protected_file_area_tar_file.png)

We then used a John the Ripper script called `ssh2john` to convert the id_rsa file to something that could be cracked, and proceeded to attack it to derive the passphrase:

![id_rsa Cracked](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/david_pass_cracked.png)

We were then able to use the id_rsa along with the cracked passphrase to log in as the david user. 

*david > root*

This was relatively trivial to exploit. In the david users home directory was a folder called 'bin' which contained a script called 'server-stats.sh'. This script was apparently being used run a command under `sudo` to grab the top lines from the `journalctl` output and print them back to STOUT. We can run the script without a password as well, which means that since `jounrnalctl` uses `less` as a pager to display content to the terminal, and the process that is spawned will be owned by the `root` user we can use this to escalate our privileges on this target. 

![sudo command as david user](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/interesting_file_david_sudo_command.png)

If the content for the terminal is too large to print all at once `less` is invoked to all us to scroll up and down and see all of the content:

![Invoke the less pager](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/tiny_terminal.png)

Now we can simply type `!/bin/sh` into less:

![less pager breakout](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/tiny_breakout.png)

Which gives us a shell as the root user: 

![root user shell](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/tiny_shell.png)

**Vulnerability Explanation:**

**Vulnerability Fix:**

**Severity:**

**Exploit Code:**

**Proof Screenshot Here:**

![root proof](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.165/root_shell.png)

**Proof.txt Contents:**

9aa36a6d76f785dfd320a478f6e0d906
