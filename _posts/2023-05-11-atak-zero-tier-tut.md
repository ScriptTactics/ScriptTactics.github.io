---
title: ZeroTier Config with ATAK and WinTAK
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
tags: atak zerotier tak takserver wintak
categories: atak
header:
  teaser: /assets/images/zeroTier.PNG
---

# Install CivTAK & ZeroTier One

Download [CivTAK](https://play.google.com/store/apps/details?id=com.atakmap.app.civ&pli=1) from the PlayStore

Also install [ZeroTier](https://play.google.com/store/apps/details?id=com.zerotier.one) from the PlayStore.

## ZeroTier
Create an account on the [ZeroTier](https://www.zerotier.com/) website.

Then create a new network.
![Create new network](/images/createZTNetwork.PNG)

## Network Config
![Network Config](/assets/images/ztNetworkConfig.PNG)
1. Name the network
2. Make sure it stays private.
3. Keep all other settings default

## Phone Config
Open the ZeroTier application. Then select Add Network
![ZT Blank](/assets/images/ztBlank.png)

![ZT Network Blank](/assets/images/ztNetworkBlank.png)
You should then see the screen above. Enter the 16hex character network key for your ZT network.

![ZT Network](/assets/images/ztNetwork.png)
Then click add.

## ZeroTier Auth
After your phone has connected to the network you will have to authorize the user. Go back to the ZeroTier web interface and give the peer a name/description and click the checkbox to authorize them.
![Authorize User](/assets/images/Authorize-user.PNG)

You will have to repeat this process for the WinTAK user and any other ATAK user you plan on adding to the network.

# WinTAK

## Windows ZeroTier
Download and install the [ZeroTier agent for Windows](https://download.zerotier.com/dist/ZeroTier%20One.msi).

Follow the same steps for the android device, but for windows.

![Windows ZT](/assets/images/ZTWindows.PNG)

Select `Join New Network...` then enter the 16 hex network id in the box that pops up and click save.

Switch back to the ZeroTier web console and authorize the new user.

## WinTAK Install

Download and install [WinTAK](https://www.civtak.org/2020/09/23/wintak-is-publicly-available/) from civtak.org.

After it's installed launch WinTAK. It should automatically discover the new ZeroTier interface.

Test the connection by using an ATAK device with WinTAK.

Ex:

![ATAK Device](/assets/images/atak.png)
This is a lone ATAK device.

With WinTAK up and running I can see the location of the ATAK device and the WinTAK device's location is shared as well.
![WinTAK device](/assets/images/winTAK.PNG)