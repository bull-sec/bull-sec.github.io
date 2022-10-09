# Windows Privilege Escalation 101

## User Accounts

These are the basic account(s) that you would use to log into a Windows machine.

They are a collection of settings/preferences bound to a unique identity. i.e. the username and the files linked to that account along with other preferences such as installed applications and group membership(s), all the various things that could be associated with a user account.

The local "Administrator" account is created by default during the installation process. It is not directly available to log in, similar to the `root` account on modern Linux systems, with the difference being that it can be activated using `net user administrator /active:yes` and it will become an account that appears on the login screen.

Depending on the version of Windows that is running there may be several other default User accounts such as "Guest".

## Service Accounts

Used to run services (duh), services are programs that are being ran either constantly (in the case of web servers and FTP applications) or on a scheduled basis by a dedicated account with permissions set up specifically for that account. So in the case of the `NT AUTHORITY\IUSR` account which is used to control the IIS web service, it has specific permissions which allow it to manage the web server and all of it's associated functions. 

They cannot be used to log in to Windows directly. But they are capable of granting a "user" shell, which will allow for access to the system but with generally limited capabilites and permissions.

The `NT AUTHORITY\SYSTEM` account is a default service account created during installation which has the highest privileges of any local account on Windows. It is the Windows equivalent of the "root" account and has *full* control over any application or user account on the system. In some instances it will even be a trusted account on other machines in the same network allowing for lateral movement.

There are other default service accounts that are created during the installation process, such as NETWORK SERVICE and LOCAL SERVICE, and the aforementioned IUSR.

## Groups

User accounts can belong to multiple groups, and groups can have multiple users (same as Linux). Not much else to explain here, just bear in mind that there are LOTS of groups on Windows systems, and especially when it comes to AD/Domain controllers you're going to have to get your reference sheets out and run things like `BloodHound` to map all of the interactions between the groups.

Groups allow for easier access control configuration by allowing multiple users the same level of access to a resource. We could created a group called "Tech_Admins" and give them access to restart/modify specific services and applications to allow them to perform maintenance or upgrades. 

There are two types of groups:

### *Regular Groups*

These groups have a set list of members and are created by the domain or system Administrator(s), with the exception of the default groups such as "Users" which contain ALL regular (non-service) users of a system.

### *Psuedo Groups*

These groups have a dynamic list of users that can change based on certain interactions, i.e. the "Authenticated Users" group contains a list of members who are CURRENTLY authenticated to the system.

## Resources

Windows has a concept of resources known as Objects, there are multiple types of these:

- Files/Directories
- Registry Entries
- Services

Everything is treated as an object and every object has a set of permissions attached to it. In most cases these are inherited from the parent directory but it is not allways the case and can lead to vulnerabilities we can take advantage of, and sometimes the inheritence itself is the thing we will look to take advantage of.

Whether a user has permission to perform certain actions on a resource will depend on what Groups that User is a member of and what permissions the User has.

```bash
whoami          # computername\username
whoami /logonid # Gets the SID for our user
whoami /priv    # Permissions
whoami /groups  # Group Membership
whoami /all     # All the above at once
```

## ACLs & ACEs

Permissions to access resources (*Objects*) in Windows are controlled by the ACLs or Access Control Lists for that resource.

Each ACL is made up of zero or more access control entries (ACEs)

Each ACE define the relationship between a Principal (the name Windows uses for Users and Groups) and a certain access right.

To view the ACL for a particular Object you can use the GUI to open the Advanced Security Settings menu, or you can run the following command in PowerShell:

```bash
Get-ACL <file>
```

```bash
PS C:\Users\MrBullsec\Documents> Get-Acl .\notes_checklist.md


    Directory: C:\Users\MrBullsec\Documents


Path               Owner         Access
----               -----         ------
notes_checklist.md CC1\MrBullsec NT AUTHORITY\SYSTEM Allow  FullControl...

```

or the following in CMD.exe:

```bash
icacls <file>
```

```bash
PS C:\Users\MrBullsec\Documents> icacls .\notes_checklist.md
.\notes_checklist.md NT AUTHORITY\SYSTEM:(I)(F)
                     BUILTIN\Administrators:(I)(F)
                     CC1\MrBullsec:(I)(F)
```