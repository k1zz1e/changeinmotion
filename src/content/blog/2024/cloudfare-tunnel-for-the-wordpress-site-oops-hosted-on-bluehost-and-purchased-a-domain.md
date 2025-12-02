---
title: "Cloudfare tunnel for the Wordpress site! Oops - hosted on Bluehost and purchased a domain"
description: ""
pubDate: "Dec 23 2024"
categories: ["Domain"]
tags: ["bluehost", "cloudfare tunnel", "domain setup", "lets encrypt", "proxmox backup", "really simple security"]
heroImage: "/images/posts/bluehost.webp"
---
Alright, I am going to start it off checking into the HTTPS settings. I setup a Cloudfare account and that end looks good to go.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-8.16.29 PM-1024x658.png)

Not sure if a need a public domain name to get the Wordpress site setup on the Tunnels section or simply make one up for local use. Going to setup cloudflare on the VM for starters.

```
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Add this repo to your apt repositories
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared jammy main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# install cloudflared
sudo apt-get update && sudo apt-get install cloudflared
```

I'm going to start it up investigating Apache2 settings.

```
systemctl status apache2
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-8.18.27 PM-1024x773.png)

DONT CHANGE THE URL IN THE WORDPRESS GUI!

I was investigating the Let's Encrypt SSL certificate generator and thought I needed to add a domain name in the Wordpress settings. Little did I know, I would make it inaccessible!

Luckily, I had a backup of my posts in XML and a VM backup. I rolled back the Ubuntu server to a previous version from 00:01 and then imported my Wordpress XML from earlier at 3:53. Whew! I am beginning to understand the true joy of backups. Once, I get back from my holiday trip I will enhance the server.

Need M.2 drives in each cluster for dedicated CEPH storage, RAM upgrades to 32GB, and a few Seagate EXOS 20TB drives- need to upgrade to an UNRAID or a enterprise server (2U) while I'm at it.

> I have ran out of time to execute the Cloudfare tunnel project before departure. I also accidentally purchased the domain I wanted on [Bluehost](https://www.bluehost.com/), lol.

So yeah, a few errors were made. But the goal of getting the Wordpress live in some ways went better. [Bluehost](https://www.bluehost.com/) is actually quite cheap for 12 months of site hosting. The domain [changeinmotion.tech](https://wbb.afh.mybluehost.me/) only cost about $8 on bluehost. I did look on Cloudfare for a similar domain but I didn't like the other options - and a transfer seemed to complicated to execute. Have to return to the Tunnel operation later.

## Domain Purchase

Going through this process was surprisingly simple.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-29-at-3.58.13 PM-1024x757.png)

To set up an account click on Domains - Buy a domain.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-29-at-4.01.45 PM-1024x804.png)

Simply type in a name you want to use for your site domain.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-29-at-4.02.11 PM-1024x804.png)

Then select from the list of available domains.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-29-at-4.05.49 PM-1024x804.png)

To upload my Wordpress site I ended up using a plugin called [UpdraftPlus Backup/Restore](https://wbb.afh.mybluehost.me/). I did try and use the WordPress Importer tool but it did not pull a 100% copy of my localhost site.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-29-at-4.10.10 PM-1024x804.png)

This plugin worked flawlessy. Site is live! Probably better that it's hosted externally from my local network. Additionally, Bluehost offered a free Let’s Encrypt SSL certificate from Really Simple Security. To many wins for so little investment up front.

I am still curious about the Cloudfare Tunnel - will likely investigate upon return. Having the site up is going to free up time for me to get back to important homelab work.
