---
title: "Cisco Packet Tracer - A basic LAN and the Cisco IOS CLI!"
description: ""
pubDate: "May 30 2025"
categories: ["Networking"]
tags: ["Cisco", "Cisco IOS", "CLI", "Router", "Switch"]
heroImage: "/images/posts/packettracer.jpg"
---
My oh my, have I been busy! I have been running almost 5 miles everyday, studying for the Security+ exam, testing network ideas at work, taking two college classes and somehow still finding a little time to play guitar and rest. Just look at these running stats!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/May-Running-Stats-1024x634.png)

And if your a big running nerd like me then you will appreciate my non-Cisco related plug for the ASICS Novablast 4s! I've been wearing these since their second iteration after I retired my Hoka Rincon addiction.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Novablast-4-1024x575.png)

But alas! Cisco Packet Tracer fun has been ongoing for my college course on Switching, Routing, and Wireless Essentials. This post will document my learning experience and completion of my second Project in class.

**Design the local area network**

So in my setup here we have a basic setup of a rather simple LAN design. Of course if you have no familiarity with using the Cisco IOS CLI, it's may not be as simple as it seems. We will be using 1 switch, 1 router, and 4 PCs - more specifically:

*   [CISCO 2960-24TT Switch](https://www.cisco.com/c/en/us/support/switches/catalyst-2960-series-switches/series.html) (EOL, support ended in 2019)
*   [CISCO 2811 Router](https://www.cisco.com/web/ANZ/cpp/refguide/hview/router/2800.html#cisco2811)
*   PC-A, PC-B, PC-C and PC-D (Windows PCs)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/LAN-Setup-Descriptionjpg-1015x1024.jpg)

To begin you need to click on \[Network Devices\] and select \[Routers\] - then in the center scroll over to 2811, click and add it. Right beside \[Routers\], you will select \[Switches\] and select 2960 from the center and add it.

To select your PCs click on \[End Devices\] then click the PC icon in the middle and add four of those. To rename them, double click on the PC icons and go to the "Config" tab. Their is an input box under "Display Name" - it's optional.

Using the \[Connections\] tool (lightning bolt) you can drag ethernet cables between each PC to the switch and then from the switch to the router.

Since we need specific interfaces on the switch for this project, click the triangle near the switch and reattach it to the switch again. It will then allow you to select a specific interface. The ones we are using are below - make sure to do them one at a time to avoid auto configurations you don't want. If you need to delete there is a button in the second row with an X on it.

**Interface configuration for this setup**

*   Switch to router on FastEthernet0/1
*   PC-A to switch on FastEthernet0/4
*   PC-B to switch on FastEthernet0/6
*   PC-C to switch on FastEthernet0/11
*   PC-D to switch on FastEthernet0/13

The FastEthernet titles represent physical ports or interfaces on the Cisco 2960 Switch. You can see below in the menu there are 24 switch ports, 2 gigabit ports and a console port. This can be viewed by hovering the switch icon.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Switch-Interfaces-1024x843.png)

_Try hovering others icons and see what information they display._

If you already configured your cables from the PCs you should see in the menu, each interface is populated with an Up status (4,6,11,13).

**Configuration settings for switch and router**

Our game plan is as follows:

*   Set the enable secret to “class”
*   Set the line con 0 password to “cisco”
*   Set the line vty 0 15 password to “cisco”
*   Set the MOTD to “Unauthorized access is strictly prohibited.”
*   Set logging to synchronous

Starting with the router (repeat process for switch), double click the router icon and select the CLI tab - you will be greeted with a startup screen. If you it asks about an initial configuration, type "No" and press enter.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Router-CLI-912x1024.png)

You should see user EXEC mode now:

```
 Router>
```

Then you can follow along with the step-by-step commands below to configure both the switch and the router to specifications outlined above.

```
Router>enable
Router>#conf t
Router(config)#enable secret class
Router(config)#line console 0
Router(config-line)#login
Router(config-line)#password cisco
Router(config-line)#logging synchronous
Router(config-line)#end
Router#
Router#conf t
Router(config)#line vty 0 15
Router(config-line)#login
Router(config-line)#password cisco
Router(config-line)#logging synchronous
Router(config-line)#banner motd "Unauthorized access is strictly prohibited."
Router(config-line)#end
Router#
Router#write memory
Router#
```

If your curious what all this is and are completely lost, then you are in for a lot of fun reading. I'll go over some basics line by line:

`**conf t**` is short for **`configure terminal`** - it is a command used to enter the global configuration mode. This mode allows you to make changes that affect the entire router or switch. We will cover modes below, I promise.

**`enable secret class`** is used to set a password for accessing privileged EXEC mode, using the password "class". Most of the commands above are for hardening your switch and router with passwords.

**`line console 0`** refers to the physical console port, which is typically the only console port on a router or switch.  It allows direct access to the device's command-line interface (CLI) using a cable.

**`login`** and **`password cisco`** is used to set the password for login to your router/switch. You will use this password initially to get into User EXEC Mode at the CLI.

The `**logging synchronous**` is to correct the output of messages when interacting with the CLI. Cisco IOS displays syslog messages to console users at any time, even while typing a command in the config line - this command prevents it. An example can be found here at [study-ccna.com](https://study-ccna.com/logging-synchronous-command/).

`**line vty 0 15**` refers to the virtual terminal lines (VTYs) on a Cisco device. These VTYs allow for up to 16 simultaneous remote connections via Telnet or SSH.

The `**banner motd**` is used to specify the message-of-the-day ([MOTD](https://www.cisco.com/E-Learning/bulk/public/tac/cim/cib/using_cisco_ios_software/cmdrefs/banner_motd.htm)) banner. When someone connects to the router, the MOTD banner appears before the login prompt. 

So if you go back up and re-evaluate the commands above you can see we are simply setting a password for the physical and virtual connections. To better understand some of the commands above we need to understand the different modes we are operating out of in the CLI. Let's break them down!

**Cisco Command Modes**

**U****ser EXEC mode**: This mode provides a restricted set of commands, mainly for basic monitoring, testing, and viewing system information. You cannot make any configuration changes that affect the device's operation in user EXEC mode. Example prompt:

```
 switch>
```

Subcommands for EXEC mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/EXEC-commands-1024x739.png)

**Privileged EXEC mode**: You enter this mode from user EXEC mode by typing the command `**enable**` and pressing enter. This mode is typically password protected with the `**enable password**` command (used in global configuration mode). From privileged EXEC mode you can configure the device, manage its operations, and access other configuration modes, like global configuration mode. Example prompt:

```
 switch#
```

Subcommands for Priviledged EXEC mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Priviledged-EXEC-commands-815x1024.png)

**Global Configuration mode**: You enter this mode from privileged EXEC mode by typing `**configure terminal**, **config t**,` or `**conf t**`. This mode allows you to configure parameters that apply to the entire device e.g., hostname, passwords, routing protocols. Example prompt:

```
 switch(config)#
```

Subcommands for Global configuration mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Global-configuration-commands-927x1024.png)

**VLAN configuration mode**: You enter this mode from global configuration mode by typing **`vlan <vlan-id>`**. This mode allows you to create, modify, and delete VLANs.

```
 switch(config-vlan)#
```

Subcommands for VLAN configuration mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/VLAN-configuration-commands-1024x213.png)

**Interface configuration mode**: You enter this mode from global configuration mode by typing **`interface`** type and number, e.g., `**interface Fa0/1**` . This mode allows you to configure the settings for a specific network interface (e.g., Ethernet, Serial). Example prompt:

```
 switch(config-if)#
```

Subcommands for Interface configuration mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Interface-configuration-mode-1024x612.png)

**Line configuration mode** - You enter this mode from the global configuration mode by using the `**line**` command with a specific line type e.g., **`line console 0`**, **`line vty 0 15`**. This mode allows you to configure the terminal lines, such as the console, Telnet, or SSH lines. .  Example prompt:

```
 switch(config-line)#
```

Subcommands for line configuration mode:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Line-config-commands-1024x610.png)

* * *

Overwhelmed yet? There are a ton of options within each command mode. It could take forever to cover all these in one post, so I will leave that to your research and point you to the [Cisco TAC Training](https://www.cisco.com/E-Learning/bulk/public/tac/cim/cib/using_cisco_ios_software/mod_frameset.htm) site. Hopefully getting a brief overview of these commands will stop you from getting _double vision_ on the Cisco site.

<Insert unnecessary comical humor here>

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/image.png)

_"Fill your eyes with.. double vision_." - Revenge of the Mooninites S01E08

My age is showing... I know. But Foreigner rules! So I had to drop a reference to the Aqua Teen Hunger Force episode with the Foreigner belt.

**Configure the computers**

For each of the computers we are going to configure the following IP, subnet, and gateway.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Cisco-LAN-PC-config-1024x595.png)

Back to the Cisco Packet Tracer GUI and click on PC-A:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Cisco-Packet-Tracer-PC-A-993x1024.png)

Then you can go to the config tab at the top and the FastEthernet0 menu on the left side - the enter your IP and Subnet Mask:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/PC-A-IP-Subnet-1015x1024.png)

Stay in the Config menu and go to the Settings menu on the left and you will find the Default gateway input field:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/PC-A-Gateway-1015x1024.png)

_Alternatively you can click on Desktop tab and go to IP Configuration (see below)._

Do this same path for all four PCs and configure them accordingly.

If you haven't already, click on one of the PCs and go to the Desktop tab in the GUI - you will see all lot of options; including the Command Prompt for running the `ping` command later.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/PC-A-Desktop-options--732x1024.png)

**Define the VLANs**

Now to define a few VLANs to get these computers connected:

VLAN 10 DataCustodians  
VLAN 20 ScrumMasters  
VLAN 30 DataOwners  
VLAN 99. Containment

Hopefully, you enjoy my addition of ScrumMasters to the VLAN titles. Go back to the GUI and double click on the switch, then click the CLI tab. You should be presented with two password entries this time - compliments of our previous steps. Commands to create your VLANs:

```
Switch>enable
Switch#conf t
Switch(config)#vlan 10
Switch(config-vlan)#name DataCustodians
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name ScrumMasters
Switch(config-vlan)#exit
Switch(config)#vlan 30
Switch(config-vlan)#name DataOwners
Switch(config-vlan)#exit
Switch(config)#vlan 99
Switch(config-vlan)#name Containment
Switch(config-vlan)#end
Switch#
```

To see your configured VLAN, use `show vlan brief` in Privileged EXEC mode. It should look like below.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/S1-show-VLAN-brief-1024x515.png)

_Notice the VLAN 1 default currently contains all the interfaces?_

**Switch Configuration**

As seen above the VLANs have been created but the interfaces for the switch have no been assigned. We need to assign them as seen below:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Switch-Configuration-1024x567.jpg)

Starting off in Global Configuration mode we can get our switch setup according to the photo above.

Switch(config)#  
Switch(config)#interface range Fa0/4  
Switch(config-if-range)#Switchport mode access  
Switch(config-if-range)#Switchport access vlan 10  
Switch(config-if-range)#exit  
Switch(config)#interface range Fa0/6  
Switch(config-if-range)#Switchport mode access  
Switch(config-if-range)#Switchport access vlan 20  
Switch(config-if-range)#exit  
Switch(config)#interface range Fa0/11  
Switch(config-if-range)#Switchport mode access  
Switch(config-if-range)#Switchport access vlan 30  
Switch(config-if-range)#exit  
Switch(config)#interface range Fa0/2-3,Fa0/5,Fa0/7-10,Fa0/12-24  
Switch(config-if-range)#Switchport mode access  
Switch(config-if-range)#Switchport access vlan 99  
Switch(config-if-range)#end  
Switch#

Now a simple `show vlan brief` command and you can see under Ports the interfaces are assigned to our VLANs we created above.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/S1-show-interfaces--1024x438.png)

_Notice the Containment VLAN has been assigned most of the interfaces?_

If you attempt a ping test now you may be hoping for some connectivity. However, if you Google our **Cisco 2960-24TT** switch being used in our LAN, you will see it is in fact a Layer 2 switch. Layer 2 switches operate on the **Data Link Layer** and cannot route traffic between different VLANs or subnets. We need to configure the Cisco 2811 router.

**Router configuration**

So for the router settings we will need to configure a few subinterfaces to mingle with our VLANs. If you look at the picture below you may be wondering, "what is subinterface 10?" Lets grab a quick definition:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Router-Configuration-1024x610.jpg)

Subinterfaces are logical divisions (virtual interface) of a physical interface on a router, allowing for multiple VLANs to be associated with a single physical link via 802.1Q tagging. These operate at Layer 3 - the network layer. Each subinterface has its own IP and VLAN ID, facilitating Inter-VLAN routing.

VLANs, on the other hand, are virtualized network segments within a Layer 2 (data link layer) network, used to segment traffic and improve network performance and security

Moving on to our router configuration, go back to the GUI and double click on the router - then go to the CLI tab and hit enter. You have another password challenge to login (we set this up in previous steps "cisco"). Type in enable to get to Priveledge EXEC mode:

```
  Router>enable
```

And you should see another password challange ("class"). Network hardening is great huh?!! It was the first thing we executed in this LAN - securing line console 0 (console cable port) and line vty 0 15 (virtual lines for SSH/TELNET) by adding passwords

Lets get back on track with the router configuration commands:

Router#conf t  
Router(config)#interface Fa0/1.10  
Router(config-subif)encapsulation dot1Q 10  
Router(config-subif)#ip address 192.168.10.1 255.255.255.0  
Router(config-subif)#interface Fa0/1.20  
Router(config-subif)encapsulation dot1Q 20  
Router(config-subif)#ip address 192.168.20.1 255.255.255.0  
Router(config-subif)#interface Fa0/1.30  
Router(config-subif)encapsulation dot1Q 30  
Router(config-subif)#ip address 192.168.30.1 255.255.255.0  
Router(config-subif)#exit  
Router(config)#interface Fa0/1  
Router(config-if)#no shutdown

You should a status change to "up" on FastEthernet 0/1 and the three virtual interfaces we just created. Fa0/1.10, Fa0/1.20, and Fa0/1.30. All three of those were assigned to same physical interface: 10, 20, and 30 all assigned to Fa0/1 (FastEthernet 0/1). This is done by the dot1Q network standard that supports VLANs. Which is what we selected via the `encapsulation` command. Let's go over that a bit.

Trunking, encapsulation and Dot1Q(the finale)

You may be asking yourself, "What is trunking? Uhmm, encapsulation? Dot1Q? Why does my ping still not work?" Let's start with a couple of definitions:

**Trunking** - a method for carrying traffic from multiple VLANs across a single physical link, like a cable connecting two switches. This is done by VLAN tagging via the Dot1Q network standard and encapsulation.

[Dot1Q, or IEEE 802.1Q](https://en.wikipedia.org/wiki/IEEE_802.1Q) - a widely used standard for VLAN trunking in Ethernet networks.  It's a protocol that allows multiple VLANs to be transmitted over a single physical link (trunk) by adding a 4-byte tag to Ethernet frames, i.e. VLAN tagging.

Watch the graphic to see how frames are tagged and routed below. If you look closely the trunk is carrying the ethernet frames from Switch1 to Switch2.

_Note: our LAN design has each PC on a separate VLAN and our switch is only layer 2. So it will not function in the manner seen below. That is by design_.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/VLAN-Tagging.gif)

Photo Credit: [NetworkAcademy](https://www.networkacademy.io/ccna/ethernet/vlan-trunking#:~:text=In%20order%20to%20overcome%20this,VLAN%2010%20and%20vice%2Dversa.)

[Encapsulation](https://geekflare.com/cloud/encapsulation-networking/) - encapsulation is the process of adding headers and trailers to data as it moves down the protocol stack, essentially "wrapping" the data with information needed for each layer to function.  See below - a dot1Q (IEEE 802.1Q) tag has been added to an ethernet frame:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/802.1q-header-1024x448.png)

Photo Credit: [NetworkAcademy](https://www.networkacademy.io/ccna/ethernet/vlan-trunking#:~:text=In%20order%20to%20overcome%20this,VLAN%2010%20and%20vice%2Dversa.)

Now that we have read a brief understanding of trunking and encapsulation we need to setup a trunk to carry data from our switch to the router. Then our PCs will be able to communicate with each other across the LAN. Click on the switch and go back to the CLI:

```
Switch>enable
Switch#conf t
Switch(config)#interface Fa0/1
Switch(config-if)#switchport mode trunk
Switch(config-if)end
Switch# show interfaces trunk
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Switch-Trunk-Status-1024x372.png)

You can see clearly at the top under port that Fa0/1 is now set to a trunking status with 802.1q encapsulation. If we wanted to be more specific we could instruct the Cisco IOS to restrict the trunk to only certain VLANs as seen in categories such as "Vlans allowed on trunk" - but that isn't required for this project.

Now to double check the router for research purposes. Go back to the GUI and click on the router, then the CLI tab and login. Let's run the following command:

```
 Router#show interfaces
```

Scroll down through the text and see if anything looks interesting..... did you find anything? How about the router subinterface we created earlier?

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Router-subinterface-status-1024x615.png)

FastEthernet0/1.10, FastEthernet0/1.20, and FastEthernet0/1.30 are all listed there with the encapsulation we configured, and a VLAN ID. By now your LAN should be functioning and ready for a **ping test**. Try to ping each PC one by one and see the results.

If you ping from each PC you should see success on all but one: PC-D. See here:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/PC-D-Ping-Test-Failure-1024x294.png)

If you go back up to the top and re-evaluate the steps you will see we assigned PC-D to the Containment VLAN and set FastEthernet0/13 to be part of it. That means that physical port is disabled from receiving data from the other three PCs. And this goes for all the interfaces assigned to the Containment VLAN.

If you are familiar with Security+ this would be considered a technical control - and it's a tiny step towards a hardened server environment.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/05/Server-1024x1024.jpg)
