# Introduction to macOS - Gatekeeper

In the last blogpost I discussed [how Apps are structures](https://github.com/yo-yo-yo-jbo/macos_app_structure) in a nutshell.  
At first it seems very promising to an attacker - let's package our payload in a nice directory structure and send it to our target!  
This idea sounds easy at first since:
- We fully control the App's Icron, we can make it seem like a document or a legitiamte App.  
- The macOS UI completely hides the `.app` extension of the directory.
