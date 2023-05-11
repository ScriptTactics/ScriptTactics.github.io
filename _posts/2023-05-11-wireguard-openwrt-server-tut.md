---
title: OpenWRT Wireguard Server Tutorial
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

{% include video id="TQxwqY-m30Y" provider="youtube" %}

# OpenWRT Wireguard Server
## Update Dependencies
```bash
opkg update
```

## Install Wireguard
```bash
opkg install wireguard-tools luci-app-wireguard
```
## Key Generation
```bash
umask u=rw,g=,o=
wg genkey | tee wgserver.key | wg pubkey > wgserver.pub 
wg genpsk > wg.psk
```

## Wireguard Interface Config
```bash
WG_IF="wg0"
WG_PORT="51820"
WG_ADDR="10.0.0.1/24"
```

## Firewall Config
```bash
WG_KEY="$(cat wgserver.key)"
WG_PSK="$(cat wg.psk)"
WG_PUB="$(cat wgserver.pub)"
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci rename firewall.@forwarding[0]="lan_wan"
uci del_list firewall.lan.network="${WG_IF}"
uci add_list firewall.lan.network="${WG_IF}"
uci -q delete firewall.wg
uci set firewall.wg="rule"
uci set firewall.wg.name="Allow-WireGuard"
uci set firewall.wg.src="wan"
uci set firewall.wg.dest_port="${WG_PORT}"
uci set firewall.wg.proto="udp"
uci set firewall.wg.proto="udp"
uci set firewall.wg.target="ACCEPT"
uci commit firewall
/etc/init.d/firewall restart
```

## Network Config:
```bash
uci -q delete network.${WG_IF}
uci set network.${WG_IF}="interface"
uci set network.${WG_IF}.proto="wireguard"
uci set network.${WG_IF}.private_key="${WG_KEY}"
uci set network.${WG_IF}.listen_port="${WG_PORT}"
uci commit network
/etc/init.d/network restart
```


# Adding a Peer
There are 2 ways to go about this in OpenWRT

Option 1: You can configure the peer on the peer itself and share the keys.

Option 2: You can generate all the keys on the server, show a QR code and then the peer can scan that to get all the information that’s required. You must then remove the private key from the server so that the peer is the only one with knowledge of its private key.


## Option 1
some text

## Option 2

Install qrencode
```bash
opkg update
```
```bash
opkg install qrencode
```

## Setup Peer
Then go to Network -> Interfaces -> wg0 -> edit -> peers -> edit peers

![Option 2 Peer](/assets/images/option2-peer.PNG)
1. Set the description of the peer
2. Click “Generate new key pair” to create both the private and public key.
3. Click “Generate preshared key”
4. Allowed IPs: 10.0.0.2/32 → this should be unique per peer. Make sure you press the + to apply the IP
5. Click Generate Configuration

![Option 2 QR](/assets/images/option2-qr.PNG)
1. All the values here should be pre-populated.
2. You can scan the QR code in your wireguard app to get the config.
3. After you’ve successfully received the configs you will need to delete the private key from the previous screen.

## Android Config Peer 2
