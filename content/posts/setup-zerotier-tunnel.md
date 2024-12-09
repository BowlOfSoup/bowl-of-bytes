---
date: '2024-12-09T00:08:00+01:00'
draft: false
title: 'Using ZeroTier as VPN-tunnel'
tags: ["VPN", "Linux", "Homelab", "Synology", "NAS"]
cover:
  image: "images/setup-zerotier-tunnel.jpg"
  alt: "image-setup-zerotier-tunnel"
  caption: ""
  relative: false
showToC: true
---

Sometimes when working with a homelab and VPN's you just want to skip all the config and just create a tunnel 
to your home network and/or (Synology) NAS. Read how ZeroTier gets this done! 
<!--more-->

### What is ZeroTier?
While [WireGuard](https://bowlofbytes.com/posts/setup-wireguard-vpn/) is a great VPN solution, it requires a bit of setup and configuration.

ZeroTier is a peer-to-peer software-defined networking (SDN) solution that creates secure virtual networks over the internet.
It acts like a LAN regardless of physical location. It's very easy to set up and even runs on Synology NAS devices.

Sign up for an account at [ZeroTier.com](https://www.zerotier.com/) and create a network. You will get a network ID that you can use to connect to.

I won't go into details on how the management screens work for ZeroTier, but it should be straightforward.

### Installation on a jump box
A jump box is a server that is used to access and 'jump to' devices, e.g. a different server not directly
accessible from the outside. I'm using my Raspberry Pi as a jump box.

<span style="color: #E83D23;">→</span> Login to the jump box and install ZeroTier.
```bash
curl -s https://install.zerotier.com | sudo bash
```
<span style="color: #E83D23;">→</span> Check the status and join your network:
```bash
sudo zerotier-cli status
sudo zerotier-cli join YOUR_NETWORK_ID
sudo zerotier-cli listnetworks
```

Now it should say something like 'no access', this is because you didn't authorize this 'member' yet.
We'll get to that later.

### Installation on a Synology NAS
I've got this directly from their website: [ZeroTier on Synology](https://docs.zerotier.com/synology/).
Follow this guide to the letter. Spoiler: You will need to enable SSH on your NAS (check the port!) and install Docker (Package center).

### Authorize members (devices) to the network
You've now enabled ZeroTier on your devices, but they both need to be authorized. Go to the ZeroTier configuration page.
The top collapsible should say "Members". There is a refreshable list of members that are connected to the network.
Select the ones you want to authorize (checkbox) and click the button "Authorize".

Repeat the status checks on your jump box to see if the box is authorized. It should state `OK PRIVATE`.

Now if you want to verify if it works, install ZeroTier on your phone or laptop and join the network (don't forget to authorize!).
Now check if you can reach one of the other connected devices.

If you want to specify IP addresses in your network, play with the settings in the 'Advanced' section on the ZeroTier configuration page.
