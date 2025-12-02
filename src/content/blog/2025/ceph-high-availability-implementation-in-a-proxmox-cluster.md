---
title: "CEPH & high availability implementation in a Proxmox cluster!"
description: ""
pubDate: "Jan 11 2025"
categories: ["Proxmox"]
tags: ["CEPH", "Cluster", "EliteDesk", "HA", "High Availability", "HP", "OSD", "TeamGroup"]
heroImage: "/images/posts/cephproxmox.webp"
---
If you look back on one of my earlier [posts](https://wbb.afh.mybluehost.me/wp-admin/post.php?post=37&action=edit) on CEPH you can see it did not go so well. I didn't really understand how it functioned in Proxmox. I knew it was a distributed storage and allowed for High Availability across the cluster but wasn't certain on the install process. After a bit of research I discovered CEPH is in fact very greedy of it's storage host. So in order to proceed on this quest for knowledge an investment had to be made... and not just time.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/20250106_161632-3-768x1024.jpg)

The TeamGroup MP33 SSD have a rating of 600TDW. Most recommend enterprise drives for CEPH, but these will be fine for my homelab.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/20250106_162044-768x1024.jpg)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/20250106_162939.jpg)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/20250106_162945-edited-1-scaled.jpg)

I got lucky on the EliteDesks I purchased. Two had Crucial 16TB sticks and two had another off brand. So I was able to match up brands in each - and got away with only buying to packs of Team Group RAM. Upgrades were complete! 32GB of RAM per node. My Windows Server will be so happy!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/20250106_173029-1024x999.jpg)

Each seemed to require and initial boot and then a reset to adjust to the new hardware. But then each came up without issue on Proxmox.

## CEPH Install

Going back in to the CEPH menu on each node - three Monitors were selected and one Manager. (I need to look into the Metadata Server function still)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.21.48 PM-1024x415.png)

Then each node had to have an OSD set up on it. Additionally you should read up on [bluestore](https://docs.ceph.com/en/latest/rados/configuration/bluestore-config-ref/). It's the OSD Type used by CEPH to read/write raw data to the drives.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.21.01 PM-1024x396.png)

After that we create a Ceph Pool in Proxmox. By default pools have a replication factor of 3 and are created using the cluster's pre-defined [CRUSH](https://docs.ceph.com/en/latest/rados/operations/crush-map/) rule.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.21.33 PM-1024x415.png)

Initially after setting everything up CEPH did not recognize the full capacity of the drives. But over a short period of time it change from a little over 1TB to 3.73TB, I believe this was due to the [autoscaler](https://docs.ceph.com/en/latest/rados/operations/placement-groups/) - still more than I expected to have overall. CEPH takes up a good chunk of available space for replication.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.22.37 PM-1024x474.png)

3.73TB of open CEPH storage available.

> CEPH is good to go! So what does it do? Well, like I said - it's distributed storage. :D

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.39.43 PM.png)

If you didn't notice already. The CEPH pool we created above is listed under each node. And is selectable as storage for LXCs and VMs in the Proxmox cluster. We still have a few more steps.

## LXC/VM configuration

So for any LXCs or VMs already installed you can simply "Move Storage" under "Disk Action" This the benefit (CEPH and high availability) in a cluster environment - utilizing the CEPH pool as a distributed storage for all the containers and VMs. This allows them to migrate across the cluster in case a host node goes down.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-11-at-12.06.01 AM-1024x430.png)

A little patience after processing the move. Some VMs can take 3-5 minutes to move from local storage to the CEPH pool. (For all newly created VM/LXCs you can select the CEPH storage initially upon setup)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.59.05 PM-1024x558.png)

You can then check under your Datacenter and see all the newly consumed space on your CEPH pool.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-11-at-12.10.35 AM-1024x484.png)

There is a lot of incredible animated pictures here at [habr.com](https://habr.com/ru/articles/313644/) as well as a solid article on how CEPH functions in a decentralized network. The site is written in Russian (it's a US based .com - and I ran a WHOIS - appears to be safe), so turn on your translator. The picture below is one of many excellent graphics that shows how the replicated data is rebuilt when a node is downed.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Ceph-Cluster-2.gif)

And just in case you forgot which CEPH you installed, you can run on the console of each node: (It's also in the subscription section of the nodes repository)

```
ceph --version
```

```
ceph version 19.2.0 (3815e3391b18c593539df6fa952c9f45c37ee4d0) squid (stable)
```

## High Availability

This step is surprisingly simple but crucial to the proper function of the cluster. Going to the Datacenter at the top of the node list - then scrolling down to HA, you will find Groups. Click create and select all the nodes in your cluster and check the nofailback box.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-10-at-11.51.08 PM-1024x539.png)

> The [nofailback](https://pve.proxmox.com/wiki/High_Availability#_configuration) options is mostly useful to avoid unwanted resource movements during administration tasks. For example, if you need to migrate a service to a node which doesn’t have the highest priority in the group, you need to tell the HA manager not to instantly move this service back by setting the nofailback option.

If you read through the documentation this avoids any chaos of LXC/VMs migrating around in mass after recovery from a failure. Last step is to add any resources you wish to be maintained in a high availability state. You can see below I have added a few LXCs; [Wallabag](https://wallabag.org/), [Homepage](https://gethomepage.dev/) and [Radarr](https://radarr.video/). (two of those I haven't wrote about yet, check em out!)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-11-at-1.35.06 PM-1024x803.png)

And now you should be up and configured with CEPH and High Availability on your cluster.

## Testing High Availability

Since my rack is accessible, I chose to pull the cable out of node3, which is running a Radarr LXC. After a few minutes you will see Proxmox begin to ramp up a copy of the LXC in the Tasks log at the bottom. It then sends the LXC to another node assigned to the high availability group in the cluster.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Before-After.png)

So as you can see CEPH and high availability are functioning well. Performance is not miserable but no were near what you would expect for a production environment. Can you imagine how this would run with a high-end cluster, 10GBPS backplane and proper aggregation switches. A couple minutes down to seconds? I'm really curious about that now. Time for more research!
