---
title: "LXC NFS mount, Proxmox VE Helper-Scripts, and Jellyfin"
description: ""
pubDate: "Feb 18 2025"
categories: ["Proxmox"]
tags: ["fstab", "Jellyfin", "lxc", "NAS", "NFS", "proxmox", "Proxmox VE Helper"]
heroImage: "/images/posts/proxmoxscripts.jpg"
---
Well, this has been a slow process getting here - I have tried to get the LXCs to share the NFS mount from the host NODE for a while. I did manage to bypass the problem using normal VMs in Proxmox ([see post](https://wbb.afh.mybluehost.me/nfs-mount-in-linux-via-fstab-radarr-sonarr-prowlarr-how-many-arrs-there/)). You can see an early failure below from before I understood the formatting of [fstab](https://unix.stackexchange.com/questions/247311/what-are-all-the-spaces-in-the-etc-fstab-for) and NFS volume name schemes.

![](/images/posts/2025/02/Screenshot-2025-02-17-at-4.57.40 PM.avif)

**THE FAILURE**

Getting the LXC to mount an NFS volume turned out to be quite simple (after countless hours of trial and error, lol). Let's get to it!

Create the location for your share:

```
mkdir /media/synology
```

Then add the volume the correct way to your host NODEs fstab:

![](/images/posts/2025/02/Screenshot-2025-02-17-at-5.14.52 PM.avif)

Next you need to configure your NAS/server with the IP address for the Proxmox host NODE which will access the directory being mounted. Since I am using a Synology NAS it looks like this:

![](/images/posts/2025/02/Screenshot-2025-02-17-at-5.19.46 PM.avif)

You can confirm access on the host NODE by going to the directory:

```
cd /media/synology && ls
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-5.26.35 PM.avif)

Confirm permissions on the "synology" directory:

```
ls -la
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-5.29.48 PM.avif)

Now we need to take a pause on setting up the NFS volume to create our Jellyfin LXC and run our [Proxmox VE Helper](https://community-scripts.github.io/ProxmoxVE/) script.

* * *

## **Proxmox Community Scripts and Jellyfin!**

Now I am not advocating away from manually installing anything, but for experience sake, these [**scripts**](https://github.com/community-scripts/ProxmoxVE) on GitHub have to be investigated. If you go to the "[ct](https://github.com/community-scripts/ProxmoxVE/tree/main/ct)" directory and scroll down you will find various scripts including [Jellyfin](https://github.com/community-scripts/ProxmoxVE/blob/main/ct/jellyfin.sh). On the Proxmox NODE (the LXC host) we will do a quick grab of the raw file:

```
# curl -O https://raw.githubusercontent.com/community-scripts/ProxmoxVE/refs/heads/main/ct/jellyfin.sh && ls
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.17.55 PM.avif)

The jellyfin.sh script should contain the following:

## Jellyfin script from Github

```
#!/usr/bin/env bash
source <(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/misc/build.func)
# Copyright (c) 2021-2025 tteck
# Author: tteck (tteckster)
# License: MIT | https://github.com/community-scripts/ProxmoxVE/raw/main/LICENSE
# Source: https://jellyfin.org/

APP="Jellyfin"
var_tags="${var_tags:-media}"
var_cpu="${var_cpu:-2}"
var_ram="${var_ram:-2048}"
var_disk="${var_disk:-16}"
var_os="${var_os:-ubuntu}"
var_version="${var_version:-24.04}"
var_unprivileged="${var_unprivileged:-1}"

header_info "$APP"
variables
color
catch_errors

function update_script() {
  header_info
  check_container_storage
  check_container_resources
  if [[ ! -d /usr/lib/jellyfin ]]; then
    msg_error "No ${APP} Installation Found!"
    exit
  fi

  if ! grep -qEi 'ubuntu' /etc/os-release; then
    msg_info "Updating Intel Dependencies"
    rm -f ~/.intel-* || true
    fetch_and_deploy_gh_release "intel-igc-core-2" "intel/intel-graphics-compiler" "binary" "latest" "" "intel-igc-core-2_*_amd64.deb"
    fetch_and_deploy_gh_release "intel-igc-opencl-2" "intel/intel-graphics-compiler" "binary" "latest" "" "intel-igc-opencl-2_*_amd64.deb"
    fetch_and_deploy_gh_release "intel-libgdgmm12" "intel/compute-runtime" "binary" "latest" "" "libigdgmm12_*_amd64.deb"
    fetch_and_deploy_gh_release "intel-opencl-icd" "intel/compute-runtime" "binary" "latest" "" "intel-opencl-icd_*_amd64.deb"
    msg_ok "Updated Intel Dependencies"
  fi

  msg_info "Updating Jellyfin"
  if ! dpkg -s libjemalloc2 >/dev/null 2>&1; then
    $STD apt install -y libjemalloc2
  fi
  if [[ ! -f /usr/lib/libjemalloc.so ]]; then
    ln -sf /usr/lib/x86_64-linux-gnu/libjemalloc.so.2 /usr/lib/libjemalloc.so
  fi
  $STD apt update
  $STD apt -y upgrade
  $STD apt -y --with-new-pkgs upgrade jellyfin jellyfin-server
  msg_ok "Updated Jellyfin"
  msg_ok "Updated successfully!"
  exit
}

start
build_container
description

msg_ok "Completed Successfully!\n"
echo -e "${CREATING}${GN}${APP} setup has been successfully initialized!${CL}"
echo -e "${INFO}${YW} Access it using the following URL:${CL}"
echo -e "${TAB}${GATEWAY}${BGN}http://${IP}:8096${CL}"
```

```
#run the bash command to execute the script
bash jellyfin.sh
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.21.43 PM.avif)

These scripts are interactive and neat!

Follow through the menus and pick your desired settings as required. Then you will land at the start screen for the script (the install takes a few minutes):

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.32.16 PM.avif)

After you successfully run the script you will notice it created a new LXC in Proxmox for you:

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.47.54 PM.avif)

At this point we will need to finish configuring the LXC with the NFS volume we mounted on the host NODE. We will edit a configuration file for the LXC on the host to share the volume. Select the .CONF file that corresponds to your Jellyfin LXC number - mine is 120.

```
cd /etc/pve/lxc
```

```
nano 120.conf
```

We need to add a line to the .CONF file for the volume on the host NODE. This will allow it to mount on the Jellyfin LXC. The line:

```
mp0: /media/synology,mp=/media/synology
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-7.12.11 PM.avif)

Reload the [systemd](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd#sect-Managing_Services_with_systemd-Compatibility_Changes) manager configuration and reset the Jellyfin LXC:

```
systemctl daemon-reload
```

```
systemctl restart lxc@120
```

You can now check on your LXC container to see if the NFS volume was shared from the host node:

```
df -h
```

or go straight there and check for the file directory of your NAS/server:

```
cd /media/synology
```

![](/images/posts/2025/02/Screenshot-2025-02-17-at-7.33.22 PM.avif)

## **Jellyfin homepage**

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.33.38 PM.avif)

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.38.14 PM.avif)

![](/images/posts/2025/02/Screenshot-2025-02-17-at-6.38.34 PM.avif)

**Click the plus by Folders:**

![](/images/posts/2025/02/Screenshot-2025-02-17-at-7.35.08 PM.avif)

Then you can select the appropriate folder and begin with all the fun of customizing Jellyfin's UI, library configuration, scanning files, etc.

![](/images/posts/2025/02/Screenshot-2025-02-17-at-7.36.10 PM.avif)

* * *

A good source of inspiration for this configuration came from the Proxmox [forums](https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/). But I found there was no real value in configuring additional user/group permissions in the LXC.

Additionally, this configuration method will work for other LXCs besides Jellyfin. Now to test and see how this configuration survives a migration across the CEPH cluster.
