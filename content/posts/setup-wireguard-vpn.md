---
date: '2024-11-15T00:08:00+01:00'
draft: false
title: 'Setting up a VPN with WireGuard'
tags: ["VPN", "Linux", "Homelab"]
cover:
  image: "images/setup-wireguard-vpn.jpg"
  alt: "image-setup-wireguard-vpn"
  caption: ""
  relative: false
showToC: true
---

In my project of building a homelab I wanted to set up a VPN to access my internal network from 
anywhere. Read on to see how I've set this up with Wireguard.
<!--more-->

### The plan
Among other devices in my network I have a Synology NAS, which I don't want to expose to the internet.
But I do want to be able to connect to the NAS and work on my homelab servers from outside my home.

![VPN Setup](/images/setup-wireguard-vpn-diagram.png "VPN setup")

I've chosen Wireguard to serve as my VPN protocol. It's quite easy to set up and has apps available
on mobile and desktop platforms. The only thing is, you will need a static public IP address to
be able to connect. If you don't have that you can either use a service like [TailScale](https://tailscale.com/)
or a Dynamic DNS (DDNS) service.

Let's dive into setting up Wireguard on a Debian based system, in my case a Raspberry Pi.

### Wireguard installation
<span style="color: #94C4FC;">→</span> Install Wireguard. 

Let's also install `qrencode` to generate a QR code for the mobile client,
which can be scanned right off the terminal to add the VPN profile to the Wireguard app.
```bash
sudo apt install wireguard qrencode -y
```

<span style="color: #94C4FC;">→</span> Create a private and public key for the server.
Don't share your private key with anyone.
```c
mkdir vpn && cd vpn
wg genkey | tee server_privatekey | wg pubkey > server_publickey
```
A private and public key will also need to be generated for the every single client that connects via Wireguard.
I tested this via my mobile phone with the Wireguard app.
    
<span style="color: #94C4FC;">→</span> Create a private and public key for the mobile client.
```c
mkdir clients && cd clients
wg genkey | tee phone_privatekey | wg pubkey > phone_publickey
```
### Server configuration
<span style="color: #94C4FC;">→</span> Edit `/etc/wireguard/wg0.conf` and add the following configuration:
```text
[Interface]
PrivateKey = <server private key>
Address = 10.7.70.5/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT; iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o eth0 -j ACCEPT; iptables -D FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <phone public key>
AllowedIPs = 10.7.70.0/24
```
* Replace <server private key> and <phone public key> with the keys you generated earlier.
* Double-check your network device name (eth0 in this case). Run `ip a` to confirm it.
* Your VPN subnet (10.7.70.0/24) must not clash with your home subnet.
* You can name the config anything instead of `wg0`. Just make sure you use that name everywhere instead.

#### Server [Interface]
|   |                                                                                             | 
|-----------|--------------------------------------------------------------------------------------------------------|
| `PrivateKey` | Paste your generated private key here.                                                                 |
| `Address`   | Subnet for your VPN, make sure this is different than your home subnet.                                |
| `ListenPort` | Port for Wireguard to work on.                                                                         |
| `PostUp`    | Enable traffic forwarding and Network Address Translation; makes sure you can access your home subnet. |
| `PostUp`    | Remove traffic forwarding and NAT, when disabling Wireguard.                                           |

#### Server [Peer]
|     |                                                                                             | 
|-------------|--------------------------------------------------------------------------------------------------------|
| `PublicKey`   | Paste your generated public key for your client, in our case 'phone' here.                             |
| `AllowedIPs`   | Allowed IP's to be assigned to client.                                                                 |

Add multiple `[Peer]` sections for each client you want to connect to the VPN.

### Client specific configuration
<span style="color: #94C4FC;">→</span> Create a client configuration file, `phone.conf` in the already created `clients` directory.
```text
[Interface]
PrivateKey = <phone private key>
Address = 10.7.70.10/32

[Peer]
PublicKey = <server public key>
Endpoint = <your IP or public domain>:51820
AllowedIPs = 10.7.70.0/24, 192.168.0.0/24
```

#### Client [Interface]
|     |                                                                                     |
|--------------|---------------------------------------------------------------|
| `PrivateKey` | Paste your generated private key here, for your client (phone). |
| `Address`    | Indicate the actual IP the client is going to be assigned to. |

#### Client [Peer]
|     |                                                                                     |
|-------------|-------------------------------------------------------------------------------------|
| `PublicKey`   | Paste your generated public key for your client, in our case 'phone' here.          |
| `Endpoint`   | Put your IP or domain here. It's your own (external) IP.                            |
| `AllowedIPs`   | IP ranges reachable by the client. In this case the VPN subnet and the home subnet. |

### Commands
<span style="color: #94C4FC;">→</span> Enable and start the Wireguard service based on you wg0 configuration file.
```text
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

<span style="color: #94C4FC;">→</span> Stop Wireguard for the `wg0` configuration. This will reset the network configuration.
```text
sudo wg-quick down wg0
```

<span style="color: #94C4FC;">→</span> Start Wireguard with the `wg0` configuration
```text
sudo wg-quick up wg0
```

### Generate a QR code to scan
To be able to use the VPN on your mobile phone, install the Wireguard app. You can generate a QR code in the
terminal with `qrencode`, open the app and scan the code to add the VPN profile.

```bash
qrencode -t ansiutf8 < clients/phone.conf
```

### Verifying and troubleshooting
#### (1) Verify via your phone or device
You should be able to connect from your phone to your internal network.
* Turn off Wi-Fi on your phone.
* Connect to the VPN.
* Try accessing your NAS or router’s web interface.

#### (2) Check if the VPN still works after a reboot.
```bash
sudo reboot 
# ...
sudo wg show
```

#### (?) My VPN server is not reachable anymore
This is mostly due to a mis-configuration in the IP ranges in the `wg0.conf` file. 
* Make sure the internal subnet actually matches with yours.
* Double check if the VPN subnet is different from the internal one.