# Introduction to macOS - Gatekeeper

In the last blogpost I discussed [how Apps are structures](https://github.com/yo-yo-yo-jbo/macos_app_structure) in a nutshell.  
At first it seems very promising to an attacker - let's package our payload in a nice directory structure and send it to our target!  
This idea sounds easy at first since:
- We fully control the App's Icon, we can make it seem like a document or a legitiamte App.  
- The macOS UI completely hides the `.app` extension of the directory.
- By default, Safari (the browser) automatically extracts archives, so a directory structure is extracted (and looks like one file).

How do we serve Apps? There are several ways to deploy an App, including `.dmg` and `.pkg` files, but for now let's host our App in an HTTP server:
```shell
zip -q -r ./Calculator.zip ./Calculator.app
python -m http.server 80
```

Unfortunately (or fortunately, depending on your point of view), if will not work... Downloading works, extracting works, but double clicking is disabled, with *no "ignore" option*:  
![My fake Calculator App is blocked](/fake_calc.png)

This is the works of the `macOS Gatekeeper`, which is what I'd like to introduce today.

## Gatekeeper - motivation and high-level description
The macOS Gatekeeper has been around for years now, but its purpose stayed the same - stop the *initial execution* of untrusted code by enforcing certain policies.  
Those policies are currently quite simple, in highlevel at least:
1. The App is well-signed.
2. The App is further notarized by Apple.

Both of those requirements are code signing requirements - the former means that a developer had to get an Apple Developer license and sign the App with his Team ID, while the latter means that the App was submitted to Apple, and by means of strict checks, it got signed ("notarized").  
The motivation behind those policies is easy to understand:
- Historically, malware authors simply purchased an Apple Developer license; while not free, they are affordable ($99 currently).
- Notarization is *not bulletproof* but stops obvious malware from spreading.

## Diving into the details
Safari and other macOS software use the API []() to download files. This makes an `extended attribute` to be saved for the downloaded file.  
Extended attributes are a way to save metadata, and are common in various filesystems, including the macOS `APFS` and `HFS+`.  
You can think of extended attributes as a dictionary (key --> data) for each file. They can be viewed and manipulated with the `xattr` command.  
In our case, the relevant exended attribute is called `com.apple.quarantine`:

```shell
jbo@McJbo ~ % xattr -l ~/Downloads/Calculator.app
com.apple.quarantine: 0083;62e09bd1;Safari;37A655F6-6704-42E5-AA69-A0169992691A
jbo@McJbo ~ %
```

The format of the contents of the `com.apple.quarantine` extended attribute is:
- Flags (hexadecimal form). `0083` means Gatekeeper checks should be enforced on this file.
- Timestamp (hexadecimal form), in [UNIX epoch](https://www.epochconverter.com/) units.
- The downloading entity (`Safari` in our case).
- A UUID that might later be referenced at a "Quarantine database".

I will not be talking about the "Quarantine database" I mentioned since currently (macOS 13.3) Apple has completely broken that database, but for completeness I will mention there is a SQLite database called `QuarantineEventsV2` that used to contain all your downloads information. You can read more about it [here](https://www.engadget.com/2012-02-14-mac-os-xs-quarantineevents-keeps-a-log-of-all-your-downloads.html) (again - not up to date).  
I, at least, see Gatekeeper as equvalent to the `Windows Mark-of-the-Web` (MoTW). Both are persistently saved in as file metadata (Windows uses NTFS [Alternate Data Streams](https://owasp.org/www-community/attacks/Windows_alternate_data_stream)) and used to later enforce policies (the ones on Windows involve [SmartScreen](https://support.microsoft.com/en-us/microsoft-edge/how-can-smartscreen-help-protect-me-in-microsoft-edge-1c9a874a-6826-be5e-45b1-67fa445a74c8), [Application Guard](https://learn.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-application-guard/md-app-guard-overview) and other technologies).

## What do we do about it?
As an attacker, you might want to bypass Gatekeeper. Well, if you manage to do it it's an actual vulnerability!  
Those kind of vulnerabilities are fun to discover, and you can read one I [responsibly disclosed to Apple](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) in 2022.  
However, there are some techniques you could use that "avoid" Gatekeeper without actually bypassing it with a vulnerability. Here are a few:
- Using a filesystem that does not support extended attributes. For example, USB drives that are formatted with `FAT32`, `ExFAT` or even `NTFS` will not get Gatekeeper bypass enforcement. Also, network drives, obviously.
- Using other methods to download files. Those include `curl`, `wget` and others - they simply download files without using Apple's API.
- Removing the extended attribute. Can be done with `xattr -d com.apple.quarantine <file>`.
- Disabling Gatekeeper completely. Must be done with root privileges: `sudo spctl --master-disable`.

Note all of these require gaining code execution on the box to begin with or *extensive* social engineering.  
I think it's fair to say Apple's Gatekeeper does stop attackers, or at least commodity malware.

## Summary
This was a short blogpost that describes one aspect of macOS security and the first line of defense (I guess).  
We will discuss other technologies in the next blogposts in the series.

Stay tuned!

Jonathan Bar Or
