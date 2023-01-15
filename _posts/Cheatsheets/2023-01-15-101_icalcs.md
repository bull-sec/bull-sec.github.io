---
layout: post
title: "Cheatsheets: ICACLs 101"
date: 2023-01-15 12:00 +0100
tags: pentesting windows cheatsheet
categories: [Cheatsheets]
published: true
---

> Understanding Windows permissions

[MSDN Link](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753525(v=ws.10)?redirectedfrom=MSDN)

```bash
icacls <FileName> [/grant[:r] <Sid>:<Perm>[...]] [/deny <Sid>:<Perm>[...]] [/remove[:g|:d]] <Sid>[...]] [/t] [/c] [/l] [/q] [/setintegritylevel <Level>:<Policy>[...]]
icacls <Directory> [/substitute <SidOld> <SidNew> [...]] [/restore <ACLfile> [/c] [/l] [/q]]
```

---

## Reading the output

See below for the full list but the main ones we're really interested in are:

- `F    (full access)`
- `RX   (read and execute access)`

Full access obviously gives us full control over the file, (Read, Write, Execute, Delete), and RX gives us Read and Execute capabilities.

And just after glancing at the others to see if anything else sticks out `WDAC` actually seems like it could be an important one to keep an eye out for as it seems to be able to control what can and can't run on a system [MSDN on WDAC](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/select-types-of-rules-to-create)

```bash
PS C:\Users\MrBullsec\Documents> icacls .\notes_checklist.md
.\notes_checklist.md NT AUTHORITY\SYSTEM:(I)(F)
                     BUILTIN\Administrators:(I)(F)
                     CC1\MrBullsec:(I)(F)
```

```bash
    SIDs may be in either numerical or friendly name form. If you use a numerical form, affix the wildcard character * to the beginning of the SID.

    icacls preserves the canonical order of ACE entries as:

        Explicit denials
        Explicit grants
        Inherited denials
        Inherited grants

    Perm is a permission mask that can be specified in one of the following forms:

        A sequence of simple rights:

        F (full access)
        M (modify access)
        RX (read and execute access)
        R (read-only access)
        W (write-only access)
        A comma-separated list in parenthesis of specific rights:
        D (delete)
        RC (read control)
        WDAC (write DAC)
        WO (write owner)
        S (synchronize)
        AS (access system security)
        MA (maximum allowed)
        GR (generic read)
        GW (generic write)
        GE (generic execute)
        GA (generic all)
        RD (read data/list directory)
        WD (write data/add file)
        AD (append data/add subdirectory)
        REA (read extended attributes)
        WEA (write extended attributes)
        X (execute/traverse)
        DC (delete child)
        RA (read attributes)
        WA (write attributes)

    Inheritance rights may precede either Perm form, and they are applied only to directories:

    (OI): object inherit
    (CI): container inherit
    (IO): inherit only
    (NP): do not propagate inherit
```
