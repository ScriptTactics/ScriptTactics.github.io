---
title: Wifi Pineapple Mk7 Wardriving
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
tags: wifi kismet mk7 pineapple openwrt hak5
categories: hacking wardriving
header:
  teaser: /assets/images/pineapple.jpg
---
# Pineapple Mk7


## Wardriving

First you'll want to install a few dependencies in order to start wardriving.

You can use this script below, or just install the packages
```bash
#!/bin/bash

opkg update

opkg install gpsd gpsd-clients kismet kismet-capture-liux-wifi
```
## Kismet setup
After installing the dependencies you will need to modify the kisment.conf file in `/etc/kismet/kismet.conf`

add this line in the `GPS` section: `gps=gpsd:host=localhost,port=2947`

## GPS Options

There are 2 options that you can use for your GPS module.
1. A USB GPS dongle
2. Android phone over Wi-Fi with GPSd Forwarder

## Option 2

Follow the steps below to setup wardriving with an Android Phone.

### GPSd
---
1. Download GPSd Forwarder for Android off of [Google Play](https://play.google.com/store/apps/details?id=io.github.tiagoshibata.gpsdclient) or [F-Droid](https://f-droid.org/packages/io.github.tiagoshibata.gpsdclient/)
2. Open the app and set the IP address to the address of your pineapple. (Default is 172.16.42.1)
3. Set the port to 9999

After you've donwloaded and setup the app. Connect your Android phone to the Pineapple via the management Wi-Fi.

### Wardriving Script
---
The following script assumes 2 things.

1. You do not have the 5Ghz wifi module for the pineapple.
2. You are using GPSd for your GPS forwarding with an Android Phone.

```bash
#!/bin/bash

pkill gpsd # kill any current gpsd process
pkill kismet # kill any kismet process
ifconfig wlan1 down # bring down wlan1
ifconfig wlan2 down # bring down wlan2
#ifconfig wlan3 down # if you have the 5GHz module oncomment this line

gpsd udp://172.16.42.1:9999 # Set the GPSD input from the android phone

sleep 1 # wait for a lock
#gpspipe -w | grep -qm 1 '"mode":3'

UTCDATE=`gpspipe -w | grep -m 1 "TPV" | sed -r 's/.*"time":"([^"]*)".*/\1/' | sed -e 's/^\(.\{10\}\)T\(.\{8\}\).*/\1 \2/'`
# SET THE PINEAPPLE'S CLOCK
date -u -s "$UTCDATE"

# LAUNCH KISMET DAEMON
# If you have the 5Ghz module you will need to add wlan3 to the config
iwconfig wlan1 mode Monitor
iwconfig wlan2 mode Monitor 
kismet -p /root/kismetlogs -t wardirve --override wardrive -c wlan1 -c wlan2
```

Copy this script to your Wifi Pineapple. After you've copied it over and your Android phone is streaming GPS information over wifi you can begin this script.

`chmod +x <filename>` to make the script executable.

`./<filename>` to run the script

## Kismet UI
If you want to modify other settings in Kismet you can start the service by typing:
```bash
kismet
```
in the terminal

Then navigate to `172.16.42.1:2501`.