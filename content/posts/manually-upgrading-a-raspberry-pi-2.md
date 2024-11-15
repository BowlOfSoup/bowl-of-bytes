---
date: '2024-11-12T00:13:00+01:00'
draft: false
title: 'Manually upgrading a Raspberry Pi 2'
tags: ["Raspberry", "Linux"]
cover:
  image: "images/manually-upgrading-a-raspberry-pi-2.jpg"
  alt: "image-manually-upgrading-a-raspberry-pi-2"
  caption: ""
  relative: false
---

I needed to dust off my trusty Raspberry Pi 2 for a new project, only to discover its Raspbian version was ancient history. 
With no SD-card reader in sight, I took the road less traveled: a manual upgrade adventure.
<!--more-->

I've set out on a quest to build myself a homelab, with some network infrastructure. 
I have an old Raspberry PI 2, with 1GB of RAM and a ARMv7 1200Mhz processor, which I indent to use as a VPN gateway. 

I booted up the Raspberry, and used the recovery mode to reset the OS to its original, which was Raspbian Jessie.
With Jessie officially retired and brimming with potential security risks, upgrading to the shiny new Raspbian Bookworm 
became my quest.

I didn't have an SD-card reader, so I couldn't just flash a new image to the SD-card. No problem! We can just manually upgrade.

Since Raspbian installs with a GUI, lets get rid of that first. With `Ctrl`+`Alt`+`1` we can switch to a terminal.
Login with `pi` and password `raspberry`. You should change this later ;)

```bash
# Remove GUI packages to free up space and simplify the system.
sudo apt purge --auto-remove \
raspberrypi-ui-mods lxpanel lxappearance openbox lightdm xserver* \
libreoffice* gvfs* desktop-base pix* gnome* gpicview xarchiver \
chromium* x11-xserver-util xserver-xorg libx11-* wolfram-engine xarchiver \
xfont* desktop-base openjdk-11-jdk libgtk2* libgtk* imagemagick* \
gnome-icon-theme adwaita-icon-theme qt-* qt5* xpdf* *wayland*
```

⚠️️ This command aggressively removes GUI-related packages and dependencies. Double-check the list to ensure you’re not 
removing something critical for your setup.

Now for the upgrading process, upgrading directly to the newest version is not recommended because of the huge amount of changes in dependencies. 

Since we're working with _Jessie_ we can't use the configured repository, since it's no longer available.

<span style="color: #B8BB25;">→</span> Remove all other lines, and add this one to `/etc/apt/sources.list`:
```text
deb http://legacy.raspbian.org/raspbian/ jessie main contrib non-free rpi
````

<span style="color: #B8BB25;">→</span> Run the following commands to update all the _Jessie_ packages:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

### To Stretch
<span style="color: #B8BB25;">→</span> Now we can upgrade to _Stretch_. Change `/etc/apt/sources.list` to this line:
```text
deb http://legacy.raspbian.org/raspbian/ stretch main contrib non-free rpi
```

<span style="color: #B8BB25;">→</span> Replace `jessie` with `stretch` in the `/etc/apt/sources.list.d/*.list` files, and upgrade away:

```bash
sudo sed -i 's/jessie/stretch/g' /etc/apt/sources.list.d/*.list
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

### To Buster
<span style="color: #B8BB25;">→</span> Repeat the same to upgrade to _Buster_; in `/etc/apt/sources.list`:
```text
deb http://legacy.raspbian.org/raspbian/ buster main contrib non-free rpi
```
<span style="color: #B8BB25;">→</span> Then run:
```bash
sudo sed -i 's/stretch/buster/g' /etc/apt/sources.list.d/*.list
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

### To Bullseye
<span style="color: #B8BB25;">→</span> Again, repeat the same to upgrade to _Bullseye_; in `/etc/apt/sources.list`:
```text
deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
```
<span style="color: #B8BB25;">→</span> Then run:
```bash
sudo sed -i 's/buster/bullseye/g' /etc/apt/sources.list.d/*.list
```

To upgrade to _Bullseye_, we manually have to remove the `ui` repo from the `*.list` files, since it's not available anymore.   

<span style="color: #B8BB25;">→</span> Edit every file in `/etc/apt/sources.list.d/` and remove the `ui` string.

<span style="color: #B8BB25;">→</span> Now run the following, and pay attention to installing `gcc-8-base`:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install gcc-8-base
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

### To Bookworm!
And lastly, we can upgrade to _Bookworm_. In `/etc/apt/sources.list`:
```text
deb http://raspbian.raspberrypi.org/raspbian/ bookworm main contrib non-free rpi
```
<span style="color: #B8BB25;">→</span> Now for the last time :)
```bash
sudo sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list.d/*.list
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
sudo reboot
```

And there you have it! A Raspberry Pi 2, manually upgraded to the latest version of Raspbian.
Verify this by running `cat /etc/os-release`, and you should see:

```text
PRETTY_NAME="Raspbian GNU/Linux 12 (bookworm)"
```