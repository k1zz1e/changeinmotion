---
title: "Windows Server & CEPH Install.  Silly ISO problem resolved."
description: ""
pubDate: "Dec 20 2024"
heroImage: "/images/posts/notes2.webp"
---
Lets us begin again!

I don't really know why, but I have had the most annoying experience getting Windows Server to install on my Proxmox Cluster. WS22 refused to acknowledge the VirtIO drivers I was using and after trying multiple versions of the drivers my cluster decided to take a dirt nap on elle3 during an upgrade to Proxmox 8.3. And to add on to that frustration, while learning how to modify files on the backend to bring it back online - I knocked out elle1. :(

\# apt-get update  
\# apt-get dist-upgrade  
reboot now

apt update && apt full-upgrade  
\*\*\*\*\*CRASH\*\*\*\*\*

I took a bit of a break here and got lost editing configurations to understand what was happening to this elle1.

nano /etc/hosts  
nano /etc/resolv.conf  
nano /etc/pve/corosync.conf  
cd /etc/pve/nodes/  
  
Nothing was bring communication of this node back to the cluster. It wasn't an IP / Hostname issue, but in the process I did discover my PiHole was blocking outgoing requests from the cluster VLAN.

search lan  
nameserver 192.168.1.9

I had to add the Cloudfare DNS - nameserver 1.1.1.1 to /etc/resolv.conf for the cluster nodes to reach out for updates. That explains why the nodes were not all wanting to execute the upgrade.

elle3 node had been acting up quite a bit when attempting to install VMs on it. I thought it was heat at first - but I think it was just instability. I wiped elle3 and brought it back to the cluster. Proxmox however still showed elle3 as being offline in GUI - and no clear way to remove it from the settings. "Join information" was grayed out.

```
root@pve1:~# pvecm nodes

Membership information
----------------------
    Nodeid      Votes Name
         1          1 elle1
         2          1 elle2
         3          1 elle3
         4          1 elle4 (local)
```

```
pvecm delnode elle3
```

Boom - fixed. Then I got immediately got brave and decided to start installing CEPH on every node. I don't even understand CEPH yet - what could go wrong!! I setup 3 monitors elle2, elle3, elle4 - and now what is going on? Not entirely sure.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-19-at-8.27.14 PM.avif)

No excitement, other than this HEALTH\_WARN and a "OSD cound 0" in the summary. Wonder what I am missing a recklessly setup a Pool. I see it populated on all the nodes as "IDKPool" and then another HEALTH\_WARN popped - "Reduced data availability." I don't think I have the bandwidth for CEPH tonight.

ABORT!

Returning back to my first mission (ADHD is a monster). Windows Server 2025! I must have tried to create this VM 5 times before realizing I had the wrong ISO. There are so many different versions. After grabbing the correct copy of the evaluation ISO I was good to go. Install complete.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-19-at-8.37.07 PM.avif)

Now what to do with Windows Server 2025? My real goal was to pentest, explore features, and Active Directory configurations. These EliteDesk 800 G4s don't have virtualization features to accommodate Hyper-V. I may have to build out a new machine for that later.

Interesting - upon investigation it seems Windows Server 2025 VM is not connected to the Internet. Likely another issue with the VirtIO drivers. Time for a break... !!!

I want to restart project Ubuntu Server VM template. Then I can spin up servers on the fly for K8S experiments.... err maybe investigate LXC containers. :D
