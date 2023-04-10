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






