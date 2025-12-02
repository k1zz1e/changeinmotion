---
title: "Homepage.dev - what took so long? Terminal apps!!"
description: ""
pubDate: "Jan 4 2025"
tags: ["bash", "docker", "docker compose", "homepage", "lxc", "proxmox", "warp", "wave"]
heroImage: "/images/posts/homepagedev.webp"
---
I've seen this before, but I was so distracted learning about Kubernetes and focusing on my job, I just never got around to it. No longer! I present to you, [homepage](https://gethomepage.dev/)!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.12.57 PM-1024x899.png)

I spun up a LXC in Proxmox on node1 and then of course got **distracted**(!!) by an insanely cool terminal app watching [Christian Lempa](https://www.youtube.com/watch?v=-QlMSLIY0JU) on YouTube.

[Warp](https://www.warp.dev/) and [Wave](https://www.waveterm.dev/) - not knowing about these sooner has made me question my inner geek.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-8.52.06 PM-1024x637.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-8.54.02 PM-1024x845.png)

Just looking at that glorious transparency makes me question a return to my previous [BBLean](https://bb4win.sourceforge.net/bblean/) & [Blackbox](https://en.wikipedia.org/wiki/Blackbox) related initiatives. Customizing the X Windows System was one of my earliest passions!

So obviously because I was so hyped to use one of these I did not want to interact with the LXC through the console in Proxmox. Little did I know that the default LXC container for Ubuntu could not easily be logged in to - there are no users preset. So of course - mo' problems!

First solution is to edit the [sshd\_config](https://klabsdev.com/enable-remote-ssh-on-proxmox-lxc/) and enable the PermitRootLogin.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-9.09.26 PM-1024x959.png)

Something about authorizing a root login sounds off sirens in my head though. I will [create a user](https://dinogeek.me/EN/LXC/How-to-manage-users-and-groups-in-an-LXC-container.html) later and set permission properly as my heart is telling me - for now, I want to see this homepage alive.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-9.27.33 PM-988x1024.png)

Oh my! Look at that terminal! The transparency!! The tabs!! Back to the tasker. I need to [install Docker](https://docs.docker.com/engine/install/ubuntu/) to run homepage.

### Set up Docker's `apt` repository.

```
# Add Docker's official GPG key: 
sudo apt-get update

sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

_If you use an Ubuntu derivative distribution, such as Linux Mint, you may need to use `UBUNTU_CODENAME` instead of `VERSION_CODENAME`._

### Install the Docker packages.

To install the latest version, run:

\# sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Verify that the installation is successful by running the `hello-world` image:

```
# sudo docker run hello-world
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-9.46.19 PM-1024x887.png)

Docker is alive! Now to backup the LXC in Proxmox!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.08.01 PM-1024x452.png)

Now we got a good base to work off I can begin to tinker with the Homepage install. If it doesn't go cleanly, I will just start over from the backup.

So being this is the first time I have ever ran a .yaml file in Docker compose, I must say, I really over complicated this process in the past! I created a homepage folder after a little [TechnoTim](https://technotim.live/posts/homepage-dashboard/) read on proper location of the install.

```
mkdir homepage
```

```
touch docker-compose.yaml
```

```
mkdir config images icons
```

```
nano docker-compose.yaml
```

Then copy paste information provided on the [homepage GitHub](https://github.com/gethomepage/homepage) into the yaml - and execute it with Docker compose.

```
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    environment:
      PUID: 1000 # optional, your user id
      PGID: 1000 # optional, your group id
    ports:
      - 3000:3000
    volumes:
      - /path/to/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock:ro # optional, for docker integrations
    restart: unless-stopped
```

```
docker compose up -d
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.38.35 PM-1024x476.png)

```
docker compose ps
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.54.00 PM-1024x269.png)

Appears to be healthy and operational. A quick "ip address show" to remember the IP address for the container and then boom!

```
http://YOURLXCIPADDRESSHERE:3000/
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.55.54 PM-1024x506.png)

After I initiated the yaml and accessed the page I went back to find a few more yaml files waiting for me in /config.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-03-at-10.57.33 PM-1024x291.png)

Oh my! A world of configuration is about to begin and who knows what I am going to unleash after figuring this out.

### To be continued tomorrow! Time for sleep!
