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

# Option 1

Follow the steps belew to setup wardriving with a USB GPS device.

## GPSd

Connect your GPS USB device to your pineapple and verify it is recognized by typing:
```bash
lsusb
```

You should see the following output:

```bash
Bus 001 Device 005: ID 1546:01a7 u-blox AG - www.u-blox.com u-blox 7 - GPS/GNSS Receiver
```
You can also verify by typing `dmesg | grep ttyACM0`. The output should be the following.

```bash
[   62.694133] cdc_acm 1-1.3:1.0: ttyACM0: USB ACM device
```

To verify everything is working run the following commands:

```bash
gpsd /dev/ttyACM0
```
This tells gpsd the input source for GPS. If the command executes successfully then you can continue. Then type

```bash
gpsmon
```

Then the output below should appear.
```bash
/dev/ttyACM0                  u-blox>
┌──────────────────────────┐┌─────────────────────────────────────────────────┐
│Ch PRN  Az  El S/N Flag U ││ECEF Pos:   000000.00m  0000000.00m  0000000.00m │
│ 0   0 000   0  00 0000   ││ECEF Vel:     -0.00m/s     +0.00m/s     -0.00m/s │
│ 1   0 000   0  00 0000   ││                                                 │
│ 2  00 000  00  00 000d Y ││LTP Pos:  00.000000000°  00.000000000°    00.00m │
│ 3  00  00   0  00 000c   ││LTP Vel:    0.00m/s   0.0°   0.00m/s             │
│ 4  00  00  00  00 000d Y ││                                                 │
│ 5  00 000  00  00 000d Y ││Time: 1 01:21:24.00                              │
│ 6  00 000  00  00 000d Y ││Time GPS: 2264+ 91284.000     Day: 1             │
│ 7  00  00  00  00 000d Y ││                                                 │
│ 8  00 000  00  00 000d Y ││Est Pos Err   0.0 m Est Vel Err   0.00m/s        │
│ 9  00 000  00  00 000d Y ││PRNs:  0 PDOP:  0.0 Fix 0x00 Flags 0xdd          │
│10 000 000  00   0 0000   │└─────────────────── NAV_SOL ─────────────────────┘
│11 000 000  00   0 0000   │┌─────────────────────────────────────────────────┐
│12 000 000  00   0 0000   ││DOP [H]  0.0 [V]  0.0 [P]  0.0 [T]  0.0 [G]  0.0 │
│13 000   0 -00   0 0000   │└─────────────────── NAV_DOP ─────────────────────┘
│14 000   0 -00   0 0000   │┌─────────────────────────────────────────────────┐
│15 000   0 -00   0 0000   ││TOFF:  0.000000000       PPS:      N/A           │
└────── NAV_SVINFO ────────┘└─────────────────────────────────────────────────┘
```
## wardrive.sh script

After you've verified the GPS device is working you can setup the following wardrive.sh script.
```bash
#!/bin/bash

pkill gpsd # kill any current gpsd process
pkill kismet # kill any kismet process
ifconfig wlan1 down # bring down wlan1
ifconfig wlan2 down # bring down wlan2
#ifconfig wlan3 down # if you have the 5GHz module oncomment this line

gpsd -n /dev/ttyACM0 # Set the GPSD input from USB Device
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
# Option 2

Follow the steps below to setup wardriving with an Android Phone.

## GPSd Forwarder
---
1. Download GPSd Forwarder for Android off of [Google Play](https://play.google.com/store/apps/details?id=io.github.tiagoshibata.gpsdclient) or [F-Droid](https://f-droid.org/packages/io.github.tiagoshibata.gpsdclient/)
2. Open the app and set the IP address to the address of your pineapple. (Default is 172.16.42.1)
3. Set the port to 9999

After you've donwloaded and setup the app. Connect your Android phone to the Pineapple via the management Wi-Fi.

## Wardriving Script
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
# Exfiltration

Once you're done running the script 2 files will be generated in the `kismetlogs` folder on the pineapple.
- YYYYMMDD-15-10-31-1.wiglecsv
- YYYYMMDD-15-10-31-1.kismet

You can open the wiglecsv file in any excel or csv application. First remove the top line in the file:
```csv
WigleWifi-1.4,appRelease=Kismet202208R1,model=Kismet,release=2022.08.R1,device=kismet,display=kismet,board=kismet,brand=kismet
```

Then import it into the excel,csv application. The column headers should be:

```bash
MAC,SSID,AuthMode,FirstSeen,Channel,RSSI,CurrentLatitude,CurrentLongitude,AltitudeMeters,AccuracyMeters,Type
```

You can further extract data from this file by writing a python script to parse the CSV and group the data.
# Kismet UI
If you want to modify other settings in Kismet you can start the service by typing:
```bash
kismet
```
in the terminal

Then navigate to `172.16.42.1:2501`.

All the settings can be adjusted here.