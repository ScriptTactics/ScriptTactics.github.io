---
title: OpenWRT Wireguard Server Tutorial
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
tags: openwrt wireguard vpn
categories: self-hosted
header:
  teaser: /assets/images/openwrt.PNG
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
This option will require sending keys between both the server and the peer (in this case Android device)

## Manual Config
![Peer 1 OpenWRT](/assets/images/Peer1.PNG)
1. Set Peer Name
2. Copy `Public Key` from `Public Key` generated in the phone app.
3. Click `Generate preshared key` and send this to the Peer.
4. Allowed IPs: Set this to a unique value within the server subnet range.
5. Save

## Option 1 Android
![Option 1 Android](/assets/images/option1Droid.png)
1. Name: OpenWRT
2. Click the refresh Icon to generate the `public/private` key pair. Send the `public key` to the server so it can be inserted into the Peer config. See Slide 27 #2
3. Addresses:  Set this to the same value populated in the peer config on the server. See Slide 27 #4
4. Click Add Peer
![Option 1 Android Peer](/assets/images/option1DroidPeer.png)

5. Copy the `server public key` and paste it here
6. Pre-Shared Key. Copy the generated pre-shared key from the peer config on the server. Slide 27 #3
7. Endpoint: `public IP` of server and the port specified by the server: 51820 is the default. 
8. Allowed IPs: `0.0.0.0/0, ::/0` → All IP’s

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
![option2-qr](/assets/images/option2Droid.PNG)
![option2-peer](/assets/images/option2-droidPeer.png)
1. After scanning the QR code you should see a config similar to this.
2. You may have to add the Addresses here. It should be an IP within the subnet range specified in the server. 

Ex: 10.0.0.2/32. (It should be unique per peer)
3. All other fields should be pre-populated.

# Verify Connection

1. You may have to restart the wg0 interface in order for a connection to get established. 
2. Once you restart the interface click the toggle button on the Android app and see if a handshake has been created.
3. The `rx` and `tx` fields should start to increment
