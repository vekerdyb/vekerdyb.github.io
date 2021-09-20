---
title : SSH Proxy Tunnel on Mac
clickbait: I've Dug A Tunnel To My Server And This Is What Happened Next
notetype : feed
date : 20-09-2021
---

Sometimes I want to access my internal network at home, to access some web services that are not open to the public.

I have an SSH server running on my server, so I can pretend I'm home by doing two things on my laptop:
      
**Step 1:**
```bash
ssh -ND 1080 my.server.com
# My .ssh config includes authentication, port, usernames for my server
```

Note: this will hang while the tunnel is open. Don't expect an output.

**Step 2:**
Then in my Mac's Network Settings > Advanced > Proxies > tick SOCKS Proxy and set `localhost` : `1080`.

When done, untick SOCKS Proxy.

The above is just for the first time setup. Once done, I use this command in my terminal to create the tunnel and set the SOCKS Proxy on my Wi-Fi connection: 

```bash
networksetup -setsocksfirewallproxystate Wi-Fi on ; ssh -ND 1080 inxui.com
```

Ctrl + C to end tunneling and then to switch off the proxy: 

```bash
networksetup -setsocksfirewallproxystate Wi-Fi off
```