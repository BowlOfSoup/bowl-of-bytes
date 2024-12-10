---
date: '2024-12-10T00:08:01+01:00'
draft: false
title: 'Sync a non-static IP with a domain provider via APIs'
tags: ["Go", "Software", "Linux", "DNS"]
cover:
  image: "images/sync-non-static-ip-transip.jpg"
  alt: "image-sync-non-static-ip-transip"
  caption: ""
  relative: false
---

The next quest in my homelab adventure was to make sure my domain name always points to my home network,
even if my ISP changes my public IP address. See how I solved this with TransIP.
<!--more-->

### The challenge
My ISP does not provide be with a static IPv4 or IPv6 address. This means that my public IP address can change.
Not handy if you want to access your home network from the outside via a domain name. This can be solved by using a
DDNS (Dynamic DNS) provider. DDNS automatically updates the DNS records of your domain whenever your IP address changes. 
A popular DDNS provider is e.g. [DuckDNS](https://www.duckdns.org/)

But. Since I'm a developer I wanted to build my on tool. _Because quests without adventures are just chores_, right? 

My domain is registered at [TransIP](https://www.transip.nl/), a well known Dutch domain provider.
TransIP provides a set of APIs to manage domain settings, like DNS records. If you have an account you can go to the
[API settings](https://www.transip.nl/cp/account/api/) and generate a Key Pair to be able to access the API. You'll
get to see the private key only once, so make sure to store it somewhere safe.

### Coding the tool
I choose to code the tool in Go. Mainly because I really like Go, and there is a good library to work with Transip APIs
available on Github: [transip/gotransip](https://github.com/transip/gotransip).

You can find the implementation here: [BowlOfSoup/dynamic-ip-syncer](https://github.com/BowlOfSoup/dynamic-ip-syncer).

To fetch my public IP addresses I use [icanhazip.com](https://icanhazip.com/). This is a simple service that returns your 
IPv4 and IPv6 address in plain text. The following URLs can be used:

* https://ipv4.icanhazip.com
* https://ipv6.icanhazip.com

And as with all my Go projects, I make a fun Gopher-logo for it:

![Go logo](/images/sync-non-static-ip-transip-go-logo.png "Go logo")

Created via [gopherize.me](https://gopherize.me/).

### Deploying the tool to a server
Ofc you should clone the repository and have Go installed on your machine before attempting to build the tool.

Also, I did not build an extensive build or install script or Docker container because of time constraints. But here are the steps I took:

<span style="color: #7843E6;">→</span> (1) Build the tool

I myself build the tool for a Raspberry PI.
```bash
GOOS=linux GOARCH=arm GOARM=7 go build -o build/dynamic-ip-syncer
```

<span style="color: #7843E6;">→</span> (2) Transfer the binary to the server
```bash
ssh YOUR_USER@YOUR_HOST "sudo mkdir -p /opt/dynamic-ip-syncer && sudo chown YOUR_USER:pi /opt/dynamic-ip-syncer"
scp build/dynamic-ip-syncer YOUR_USER@YOUR_HOST:/opt/dynamic-ip-syncer/
```

**Now on the server itself**, configure the tool to run as a service. I used `systemd` for this.

<span style="color: #7843E6;">→</span> (3) Create a `.service` file with the following content:
```text
[Unit]
Description=Dynamic IP Syncer
After=network.target

[Service]
ExecStart=/opt/dynamic-ip-syncer/dynamic-ip-syncer
WorkingDirectory=/opt/dynamic-ip-syncer
Restart=always
Environment=CONFIG_PATH=/etc/dynamic-ip-syncer/config.yaml
Environment=PRIVATE_KEY_PATH=/etc/dynamic-ip-syncer/private.key
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

<span style="color: #7843E6;">→</span> (4) move the file to `/etc/systemd/system/dynamic-ip-syncer.service` and enable the service:
```text
sudo mv dynamic-ip-syncer.service /etc/systemd/system/
sudo chown root:root /etc/systemd.system/dynamic-ip-syncer.service && sudo chmod 644 /etc/systemd/system/dynamic-ip-syncer.service
sudo systemctl enable dynamic-ip-syncer
```

<span style="color: #7843E6;">→</span> (5) Make sure to create a `config.yaml` file in `/etc/dynamic-ip-syncer/` with the correct values.

<span style="color: #7843E6;">→</span> (6) Make sure to create a `private.key` file in `/etc/dynamic-ip-syncer/` with the private key you got from TransIP.

<span style="color: #7843E6;">→</span> (7) Start the service; the service will run in the background, and starts on reboot.
```bash
sudo systemctl start dynamic-ip-syncer
```

You can view the **logs** with the following command:
```bash
sudo journalctl -u dynamic-ip-syncer --since "2 hours ago"
```