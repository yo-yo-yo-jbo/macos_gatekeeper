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

