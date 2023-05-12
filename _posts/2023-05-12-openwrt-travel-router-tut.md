---
title: Wireguard OpenWRT Travel Router Tutorial
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
tags: openwrt wireguard vpn travel-router
categories: self-hosted
header:
  teaser: /assets/images/travelRouter.png
---
{% include video id="" provider="youtube" %}

# OpenWRT Wireguard Travel Router

# Pre-Requisite
The travel router will be a Wireguard Peer, so you will need an existing Wireguard server setup.
Any of the previous examples will work as the Wireguard server. This tutorial will only cover setting up the Peer, not the server, since those steps were covered in other tutorials on the site.

You will also need:
 - Raspberry Pi
 - Wireless Adapter Compatible with OpenWRT
 - Micro-SD card for OpenWRT
 - OpenWRT
 - Wi-Fi
 - Ethernet cable

# Setup
 Download OpenWRT for your Raspberry Pi. Install it onto the sd card and then follow the next steps.

1. Power on your Raspberry Pi and connect a LAN cable to the ethernet port and connect your computer to that LAN cable.

2. The router should give out a DHCP address. If not, you will have to set your IP to 192.168.1.
OpenWRT uses 192.168.1.1 by default


# OpenWRT
Once you are able to access the admin gui interface on the Raspberry Pi navigate to the wireless section.

# Radio0
Network -> Wireless

![Radio0](/assets/images/radio0.png)

1. Radio0 is the on-board wifi of the Rpi. Enable this device and then scan for wifi.
2. Find your home network and connect to it.

![Radio0 Connection](/assets/images/joinNetwork.png)

3. After you’ve found your network select “Replace wireless configuration”
4. Set the name to wwan
5. Enter the WPA passphrase of your network and then click submit.
6. You will be brought to another config page, but just click save and do not make changes
7. This will give the Rpi network access.


# Update and Install Packages
After you’ve connected the RPi to your local wifi you can update and install packages.

You will need to SSH into the Raspberry Pi and then run these commands.

To ssh run the following `ssh root@<rpi-IP>`. Repace `<rpi-IP>` with your actual IP. Usually it's `192.168.1.1`

Run 
```bash
opkg update
``` 
to download the package repository list.

Then install wireguard.
```bash
opkg install wireguard-tools luci-app-wireguard
```

Both these commands can be run in the GUI. 

## Install USB Drivers
After you’ve installed Wireguard you will also need to install USB Drivers.

Run: 
```bash
opkg install kmod-rt2800-lib kmod-rt2800-usb kmod-rt2x00-lib kmod-rt2x00-usb kmod-usb-core kmod-usb-uhci kmod-usb-ohci kmod-usb2 usbutils
```

# Radio1

![lsusb before](/assets/images/lsusb%20before.png)
Run `lsusb` in the console to see what devices are connected to the RPi. After you have the list connect your wifi card to the USB port on your RPi.

![lsusb after](/assets/images/lsusbafter.png)
Run `lsusb` again to see if a new device has been discovered. In this case the `Ralink 802.11 n WLAN` device has been found. Now you can switch back to the GUI

## Radio1 Config
Under Network→Wireless you should now see a second radio device (radio1). Click Edit to bring up the config menu.
![radio1](/assets/images/radio1.png)

![radio1Config](/assets/images/raido1.PNG)
1. Mode: Access Point
2. ESSID: Anything you want (default OpenWRT)
3. Network: wwan

Select Wireless Security.
![radio1 Security](/assets/images/radio1ConfigSecurity.png)

1. Encryption: WPA2-PSK/WPA3-SAE Mixed Mode
2. Key: This should be no less than 16 characters
3. Click Save
4. Then on the main Wireless Screen click save and apply
5. Also be sure to enable the device

# Wireguard Setup
Network → Interfaces
1. Select Add new interface

![Create new Interface](/assets/images/createNewInt.png)

1. Name: wg0
2. Protocol: WireGuard VPN

**NOTE**: If `WireGuard VPN` does not show in the protocol list you may have to reboot your raspberry pi.
![wg0 interface](/assets/images/wg0Interface.png)

## General Settings
**NOTE You will need to create a peer on your Wireguard server prior to completing this step. The tutorial assumes you have already done so.**

1. Generate new Key Pair
2. Ip Addresses: This will be the IP of the Peer you configure on your WG server.

The Public Key here will also go on your WG server as the Peer Public Key.
![wg0 Config](/assets/images/wg0Config.png)

## wg0 Peer
![wg0 Peer Config](/assets/images/peerConfig.png)

1. Description: Anything you want
2. Public Key: Server’s public key
3. Private key: leave blank
4. Preshared Key: PSK generated from server (do not generate here)
5. Allowed IP’s: `0.0.0.0/0,::/0` (allow all IPv4 & IPv6 Traffic)
6. Route Allowed IPs: `Check`
7. Endpoint Host: Public IP of your server
8. Enpoint Port: Port that wireguard is running on your server.

Once this has been completed. Click save and then click save and apply on the main Interface Page. You may have to restart the `wg0` interface to establish a connection.
# Firewall
Next we have to move back to the terminal to setup the firewall rules.

I reccommend copying the commands 1 by 1 to ensure everything is correct.
```bash
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci del_list firewall.wan.network="wg0"
uci add_list firewall.wan.network="wg0"
uci commit firewall
/etc/init.d/firewall restart
```

1. Set zone 0 to lan
2. Set zone 1 to wan
3. Remove wg0 from wan network
4. Add wg0 to wan network
5. commit changes
6. restart the firewall

# Verify
After committing firewall changes you may have to reboot the RPi to get successful connection.

Once the device comes back online you should see traffic on the Network→Interfaces page under the wg0 interface. Rx & Tx should have numbers.