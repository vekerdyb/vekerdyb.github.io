---
title : Setting up flow.ci on my Fedora server
notetype : unfeed
date : 16-10-2021
tags: 
    - home server
	- fedora
	- ci
---

https://github.com/FlowCI/docs/blob/master/en/start/index.md

After installation, open the ports:

```
firewall-cmd --add-port=2015/tcp --add-port=8080/tcp --permanent
```
