---
title: "NFS mount in Linux via fstab! Radarr, Sonarr, Prowlarr - how many 'arrs' there?"
description: ""
pubDate: "Jan 19 2025"
categories: ["Uncategorized"]
heroImage: "/images/posts/arrsnfs.webp"
---
What a journey it has been trying to get my NAS to mount in these LXCs in Proxmox. I've tried quite a bit of variations in the MOUNT command and ultimately landed on:

```
sudo mount -t nfs 192.168.1.18:/volume1/Video /media/synology
```

```
df -h
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-12.28.27 PM-1024x394.png)

I settled on this simple approach after trying to authenticate SMB and later NFS with a username/password approach for about a week. Most of that struggle was due to unprivileged LXCs in Proxmox sharing so many resources with the host node (as previously discovered with [kernel configs](https://wbb.afh.mybluehost.me/lets-cause-some-ruckus-kubernetes-in-a-proxmox-lxc-what-could-go-wrong/)). Further that confusion by adding on another layer of permissions with containers running via [Docker Engine](https://docs.docker.com/engine/) and [Portainer](https://www.portainer.io/). (A lot of fun happened this week with containers, lol)

A Reddit post tipped me off this simpler approach:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-12.33.33 PM-1024x789.png)

I had to configure, guest access to the specific folder on the NAS the containers need access to, then mount it. Which the above command does, but not automatically on boot.

## /etc/fstab

In order to mount the remote directory on startup we must modify **[/etc/fstab](https://en.wikipedia.org/wiki/Fstab)**. Then it will automatically start after any restarts or power outages. Seems simple enough, now I just got to translate the above command correctly for **fstab**. A cleaner graphic is provided by [Linuxize](https://linuxize.com/post/how-to-mount-an-nfs-share-in-linux/) if the Wikipedia example seems overwhelming.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-2.47.42 PM-1024x186.png)

That was not enough for me however. I had to see why there is an unusual amount of white space between each category. Seems based on the conversations at [Stackexchange](https://unix.stackexchange.com/questions/247311/what-are-all-the-spaces-in-the-etc-fstab-for) it's mostly for appearances and readability.

So the end result was formatted:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-2.59.47 PM-1024x204.png)

You see the lack of consistency in white space - if you appreciate organization and structure that could definitely bothersome. Now for a quick restart to test.

```
sudo reboot now
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-3.02.12 PM-1024x580.png)

Checking Sonarr reveals in fact the containers are still seeing the NFS volume mount after restart. This is great news! If only you seen the offline struggles to get to this moment! So why did I go through all the effort this week you may ask?

## Sonarr, Radarr, Jackett, qBitorrent in Windows Server 2025

I can see the appeal of why so many companies would rather pay the license fee for Microsoft Windows. The effort it took me to configure services in Windows was effortless by comparison to Linux. Many people fault Linux for it's difficulties, but often take for granted how many hours of their lives they have spent training in a Windows environment. Of course it's easier! You've practiced!

As you can see here Windows Server loves RAM:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-3.16.54 PM.png)

This coupled with a desire to maintain mobility across the cluster has been the motivation for configuring containers in their own separate LXCs. But after getting frustrated trying to pass permissions for NFS mounts through to Docker and Portainer I settled for putting media containers into a single Ubuntu Server install. I don't want to configure 5-8 LXCs.... yet!

Back to Windows Server! As you can see below the visual comfort of this **three step process** will have you installed and accessible across the LAN quickly. Run your desired .exe:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-3.46.07 PM-1024x818.png)

Configure your firewall to allow access outside the server (select Allow another app.. if necessary):

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-3.35.04 PM-1-1024x824.png)

For the Windows Server I utilized the [Samba protocol](https://en.wikipedia.org/wiki/Samba_\(software\)) / SMB / CIFS. The folder directory I got from my SMB settings in the Synology NAS under File Services > SMB.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-18-at-3.56.22 PM-1024x818.png)

And after that can go through and configure all your Sonarr/Radarr directories and get lost in their respective [configuration documentation](https://wiki.servarr.com/), haha! But you can see visually things are simpler here. You don't have to struggle through finding the unknown usage of a command modifier - or getting lost on permission related issues for days.

### TO BE CONTINUED! It's time for hockey!
