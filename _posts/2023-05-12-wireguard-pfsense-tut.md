---
title: PfSense Wireguard Server Tutorial
author_profile: false
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
tags: pfsense wireguard vpn travel-router
categories: self-hosted
header:
  teaser: /assets/images/pfSense.PNG
---
{% include video id="" provider="youtube" %}

# Install Wireguard Package
Go to System → Package Manager → Available Packages.

Search for “Wireguard”

![Wireguard Package](/assets/images/wgpkg.PNG)
At the time of making this, version 0.1.6_2 is the most up to date.
# Tunnel Config

From the menu at the top, select VPN → Wireguard.

Select [+ Add Tunnel]

![Tunnel](/assets/images/tunnel.PNG)

![Tunnel Config](/assets/images/tunnelConfig.PNG)

1. Enable tunnel
2. Set a description for the tunnel
3. Set the port to anything. (Default is 51820)
4. Click “Generate”  to create your public and private keys. 
5. Create interface addresses.

Interface addresses can be any internal IP address range. Here I chose the `10.0.0.1/24` subnet.

Meaning I can use `10.0.0.2-10.0.0.255`
See [RCF1918](https://www.rfc-editor.org/rfc/rfc1918).

Once finished, click save. Then go to Peers.

# Peer Config

![Wireguard Peer](/assets/images/wgpeer.PNG)
1. Enable peer.
2. Select the tunnel you created (in this case tun_wg0).
3. Set a Description.
4. Keep the endpoint dynamic.
5. Keep Alive: leave blank.
6. Public Key* this will be generate from your app.
7. Pre-Shared Key (optional but can provide quantum protection) Click [Generate] to create one.
8. Allowed IP’s: Set this to an address that has not been used in your subnet yet. 
Ex: 10.0.0.2/32 (means only 10.0.0.2)


# App Config: Android
![Pfsense Peer](/assets/images/pfSensePeer.png)

1. Interface Name: Pfsense or any name you prefer 
2 Click the refresh icon in the Private Key section to generate a public and private key.
3. The addresses should be the address you gave to the peer. In this case it is 10.0.0.2/32.
4. DNS Servers: You can set the DNS server to the upstream DNS on your network (usually 192.168.0.1 or 10.0.0.1) if you created a DNS for the tun0 interface, or leave this blank.
5. MTU leave empty.
6. Copy the Public Key and paste it into the Public Key section for the Peer on the server. See Slide 7, #6.

Then select “Add Peer” at the bottom
![PfSense Config](/assets/images/pfSenseConfig.png)
1. Public Key: Copy the public from the server. (Slide 6 #4)
2. Pre-shared key: Copy the pre-shared key from the server. (Slide 7 #7)
3. Persistent keepalive: Leave Blank
4. Enpoint: This should be the public IP of your server. In my case my PFsense box is a VM on my local net so the IP is 192.168.0.44. Also :51820 is the port specification we configured.
5. Allowed IPs: 0.0.0.0/0 means any IP address on IPv4 and ::/0 means any IP address on IPv6. Setting it up this way means we created a Full-Tunnel. All traffic on the device will be routed through the wireguard VPN.

# Firewall Config
Select Firewall → Rules from the main menu.

![Firewall Wan Default](/assets/images/FirewallRulesWANDefault.PNG)
By default your WAN should only have 1 rule.

Block bogon networks.

Mine has an additional rule since I do not have a LAN address on this VM. Ignore this rule.

Select “Add \/”

![Firewall Rule Settings](/assets/images/FirewallRuleSetting.PNG)
1. Action: Pass
2. Interface: WAN
3. Address Family: IPv4
4. Protocol: UDP
5. Source: any
6. Destination: WAN address
7. Destination Port Range: Custom: 51820 to Custom: 51820
8: Log Packets: Optional (can create a lot of spam)
9. Description: Wireguard – Allow WAN 51820

Then click Save. You may have to click appy changes after you save.

## Wireguard Firewall
By default there are no rules to allow Wireguard traffic anywhere. We need to create 1 rule to allow all traffic. But be warned, this tutorial does not cover how to properly segment your network and add additional security to your wireguard interface. VLANs, proper firewall rules, and network segmentation is important.

![WG Rules](/assets/images/wgRules.PNG)

Select Add to create a new rule


![Allow All](/assets/images/AllowAll.PNG)
1. Action: Pass
2. Interface: Wireguard
3. Address Family: IPv4
4. Protocol: Any
5. Source: Any
6. Destination: Any
7. Log: Optional
8. Description: Allow all
9. Save


Apply Changes after saving

## NAT Rules
Select Firewall → NAT and then Outbound to arrive at this page.
![NAT Default](/assets/images/HybridNAT.PNG)

In order to route Wireguard traffic back out the WAN we need to configure a few things in the NAT settings.

Set Outbound NAT Mode to Hybrid.

![NAT Default](/assets/images/FirewallNATOutbound.PNG)
Select “Add /\” under Mappings to create a new NAT Outbound Mapping.

![NAT Settings](/assets/images/NAT-settings.PNG)
1. Interface: WAN
2. Address Family: IPv4
3. Protocol: any
4. Source: Network – 10.0.0.1/24

Then click save. You may need to apply the changes after clicking save.

# Verify
![Verify](/assets/images/VerifyStatus.PNG)
After saving and reloading the firewall go back to VPN → Wireguard → Status. Click the down arrow on the tun_wg0 tunnel. You should see that the peer you created has successfully been created.