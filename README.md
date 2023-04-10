# Introduction to macOS - Gatekeeper

In the last blogpost I discussed [how Apps are structures](https://github.com/yo-yo-yo-jbo/macos_app_structure) in a nutshell.  
At first it seems very promising to an attacker - let's package our payload in a nice directory structure and send it to our target!  
This idea sounds easy at first since:
- We fully control the App's Icron, we can make it seem like a document or a legitiamte App.  
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

