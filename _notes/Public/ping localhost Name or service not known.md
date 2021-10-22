---
title : Fedora 33 - ping: localhost: Name or service not known
notetype : feed
date : 18-10-2021
tags: 
    - home server
	- fedora
	- network
---
`ping: localhost: Name or service not known` after updating to Fedora 33 from Fedora 31. Why?

Previously `NetworkManager` was disabled, and it got re-enabled during upgrade.

Re-disabling solved it: 
```
systemctl stop NetworkManager
systemctl disable NetworkManager
```