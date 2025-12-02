---
title: "Proxmox LXC - What can I do with these?"
description: ""
pubDate: "Dec 20 2024"
#categories: ["Proxmox"]
#tags: ["CEPH", "OSD", "proxmox", "Ubuntu", "ubuntu server"]
heroImage: "/images/posts/lxc.webp"
---

Well, off to the races! I shift objectives so quickly!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-19-at-10.44.37 PM-833x1024.png)

Well, thanks to this \[deleted\] post I quickly came to understand the process of grabbing these Linux OS templates: [https://images.linuxcontainers.org/](https://images.linuxcontainers.org/) - of course it is never quite as simple as one would hope. Manage to download and execute the container in Proxmox - however, it wouldn't start and then vanished.

That brings me to the [https://pve.proxmox.com/wiki/Linux\_Container](https://pve.proxmox.com/wiki/Linux_Container) page.

```
pveam updatepveam available 
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-19-at-10.56.36 PM-1024x1020.png)

Ah ha! Packages!

```
pveam download local <distro of choice> 
```

Emphasis on the local part! Would be much better if a had a neutral storage setup. I got the DS1520+ setup on SMB earlier but it doesn't appear any many options for storage in the cluster - don't remind me about the "IDKPool" I setup in CEPH. It's still in the GUI despite my removing it earlier.

Trying to install through the CLI is easier if you know what you are doing already. Getting an error attempting to install the container.  
storage: storage 'local' does not support container directories

\# pct create 110 local:vztmpl/ubuntu-24.10-standard\_24.10-1\_amd64.tar.zst

Apparently "local" is not container friendly. Have to use local-lvm. The following are examples I found for executing the containter through the CLI with modifiers for installing to local-lvm.  
[https://forum.proxmox.com/threads/storage-local-does-not-support-container-directories-when-creating-new-containter.68317/](https://forum.proxmox.com/threads/storage-local-does-not-support-container-directories-when-creating-new-containter.68317/)

* * *

```
pct create 100 local:vztmpl/ubuntu-24.10-standard_24.10-1_amd64.tar.zst --rootfs local-lvm:8pct create 100 local:vztmpl/ubuntu-24.10-standard_24.10-1_amd64.tar.zst --rootfs local-lvm:8 --cores 2 --net0 name=eth0,bridge=vmbr0,ip=dhcp --unprivileged 1 --password 12345 --features nesting=1
```

* * *

I didn't get to try them. Decided to attack it through the GUI on the posters suggestion. ;D

"If you're pretty new to Proxmox VE I recommend using the Webinterface for doing such things, it assists one much more than the CLI."

The premade containers from "pveam available" spin up effortlessly! I started up two of them so far. Seems they will allow for effortless Kubernetes cluster expirements. That was whole reason I ran down this rabbit hole - manually wiping Raspberri Pi often from failed K8S/K3S/Microk8s clusters got old quickly. Time for bed!

Edit post: **pveceph pool destroy IDKPool --force**  
  
Now the IDKPool is removed from the CEPH dropdown under Pools. Still showing under each node dropdown. Interesting!
