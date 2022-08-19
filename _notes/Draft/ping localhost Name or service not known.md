---
title : Fedora 33 - ping: localhost: Name or service not known
notetype : unfeed
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

It happened again later...
```
sudo systemctl enable systemd-resolved
sudo systemctl start systemd-resolved
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```
solved it I think 
From:
https://blog.technitium.com/2017/11/

Also: 
https://www.xmodulo.com/disable-network-manager-linux.html