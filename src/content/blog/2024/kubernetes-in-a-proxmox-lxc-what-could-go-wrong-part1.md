---
title: "Kubernetes in a Proxmox LXC - what could go wrong? #part1"
description: ""
pubDate: "Dec 22 2024"
#categories: ["Kubernetes", "Proxmox", "Ubuntu"]
#tags: ["K8S", "Kubernetes", "Ubuntu"]
heroImage: "/images/posts/proxmoxk8s.webp"
---

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-7.57.47 PM-1024x667.png)

Last times journey in LXC containers reveal an incredible simply way to install Linux containers with "pvecm available" and I spun up three containers. This was the sole reason behind buying four Elitedesk 800 G4s - to get away from Kubernetes installs on Raspberri Pi 4s. Way easier to learn if I don't waste a bunch of time wiping SD cards.

Lets spin up some K8S on these LXC containers!

Ubuntu 24.10 containers don't have "curl" but additionally we need to update our packges.

sudo apt update

apt install curl

```
# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Validate the binary:

```
# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

```

echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

If valid, the output is:

```
kubectl: OK
```

Install kubectl (you can see it in the current folder with ls)

```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

> Here we go again! The simple part is initiating kubectl. Typically after this I would join the nodes together and hit a brick wall. This time I will brave the kubeadm!
> 
> [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm)

As per requirements of installing kubeadm I will check the required ports for openings on port 6443 with:

```
nc 127.0.0.1 6443 -v
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-9.15.11 PM-1024x621.png)

Doesn't look good, but the guide doesn't really specify what I am looking for. Quick search reveals a few golden eggs:

```
ufw allow "port-number/protocol"
     or
ufw allow 6443
```

Since nmap is not installed on the Ubuntu containers, I attempted to nmap the LXC2 container from my MacBook at IP: 192.168.2.15 and only see the SSH port is opened. :(

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-9.21.48 PM-1024x612.png)

A more complex way I discovered was to use lsof

```
sudo lsof -i -nP | grep LISTEN
```

*   **\-i** stands for “Internet”, but basically means that we are only interested in network ports open.
*   **\-n** disable the host resolution to show the IP addresses directly (faster).
*   **\-P** to display the port numbers.
*   The **grep** command is used to filter the list with only listening ports (not the ones used by established connections, for example).

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-9.35.16 PM-1024x616.png)

Very exciting! But no port 6443 on the list. kubeadm states this as a requirement so I am not certain what is next here...

```
sudo lsof -i:22 (22 = port number)
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-9.36.29 PM-1024x616.png)

More excitement! COMMAND information relating to the port and so much more! Sadly, it did not reveal anything for 6443 but as an example I searched for port 22.

Another way to get a list of open ports:

```
sudo ss -ltn
```

*   **\-t**: Only show TCP ports (use -u to include UDP ports).
*   **\-l**: Only list ports open in LISTEN mode (the grep equivalent),
*   **\-n**: Shows port numbers, do not use their names (http, ssh, …).

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-9.37.21 PM-1024x616.png)

Additional use of the GREP command can filter results.

```
sudo ss -ltn | grep 22  (22 = port number)
```

You can also nmap your own host if you use:

```
sudo nmap -p 80 localhost
```

* * *

DISTRACTION COMPLETE! What rabbit hole. Lets get back to messing with **kubeadm**. Not so sure this port will open unless a service requests it. I already added it to the firewall.

* * *

So many issues with kubeadm install. No wonder I never got it up and running in the past. [https://github.com/kubernetes/kubeadm/issues](https://github.com/kubernetes/kubeadm/issues)

* * *

## These instructions are for Kubernetes v1.32.

1\. Update the `apt` package index and install packages needed to use the Kubernetes `apt` repository:

```
sudo apt-get update

# apt-transport-https may be a dummy package; if so, you can skip that package

sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

2\. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:

```
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.

sudo mkdir -p -m 755 /etc/apt/keyrings

# curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

3\. Add the appropriate Kubernetes `apt` repository. Please note that this repository have packages only for Kubernetes 1.32; for other Kubernetes minor versions, you need to change the Kubernetes minor version in the URL to match your desired minor version (you should also check that you are reading the documentation for the version of Kubernetes that you plan to install).

```
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

4\. Update the `apt` package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

5\. (Optional) Enable the kubelet service before running kubeadm:

```
sudo systemctl enable --now kubelet
```

The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do.

* * *

```
kubeadm init
```

Is throwing mad errors!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-10.34.40 PM-1024x889.png)

> "Failed to parse kernel config: unable to load kernel module."

Apparently, since this is an LXC and not a bare metal install, the kernel information is not loaded - as it is maintained on the host node. **kubeadm** wants this config file to work with the kernel. Solution as suggested by users in Stack Overflow is to push the config into the LXC. [https://stackoverflow.com/questions/63427047/proxmox-lxc-add-add-linux-kernel-modules](https://stackoverflow.com/questions/63427047/proxmox-lxc-add-add-linux-kernel-modules)

```
pct push $VMID /boot/config-$(uname -r) /boot/config-$(uname -r)

Where $VMID is your container id.
```

The pct command is to be used in the shell of the node to manage Linux Containers (LXC) on Proxmox [https://pve.proxmox.com/pve-docs/pct.1.html](https://pve.proxmox.com/pve-docs/pct.1.html)

Ah! New errors to investigate! Lovely! <3

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-10.41.51 PM-1024x891.png)

Appears, upon research there may be a lack of containerd in these LXCs. The new preflight warning speaks to this.

```
W1222 07:01:13.328433    1848 checks.go:1080] [preflight] WARNING: Couldn't create the interface used for talking to the container runtime: failed to create new CRI runtime service: validate service connection: validate CRI v1 runtime API for endpoint "unix:///var/run/containerd/containerd.sock": rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial unix /var/run/containerd/containerd.sock: connect: no such file or directory"
```

There is in fact no "containerd" located under /var/run.. maybe I should try to install it? Grab the URL for [containerd-2.0.1-linux-amd64.tar.gz](https://github.com/containerd/containerd/releases/download/v2.0.1/containerd-2.0.1-linux-amd64.tar.gz) here:  
[https://github.com/containerd/containerd/releases](https://github.com/containerd/containerd/releases)

```
# curl https://github.com/containerd/containerd/releases/download/v2.0.1/containerd-2.0.1-linux-amd64.tar.gz
```

Then following along with the "Getting Started" section we need to extract the tar file locally.

[https://github.com/containerd/containerd/blob/main/docs/getting-started.md](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

tar Cxzvf /usr/local containerd-2.0.1-linux-amd64.tar.gz

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-11.11.55 PM-1024x460.png)

> If you intend to start containerd via systemd, you should also download the `containerd.service` unit file from [https://raw.githubusercontent.com/containerd/containerd/main/containerd.service](https://raw.githubusercontent.com/containerd/containerd/main/containerd.service) into `/usr/local/lib/systemd/system/containerd.service`, and run the following commands:
> 
> systemctl daemon-reload  
> systemctl enable --now containerd

Oh how exciting! curl simply displays the file I am in trying to download in BASH.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-11.19.01 PM-1017x1024.png)

New command knowledge! curl the text from the URL into a file called containerd.service

```
# curl https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o "containerd.service"

cp containerd.serivce /lib/systemd/system
```

```
cd /lib/systemd/system && ls
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-11.29.00 PM-646x1024.png)

We can now see the containerd.service file located in the directory.

## Installing containerd[](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#installing-containerd)

### Option 1: From the official binaries[](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#option-1-from-the-official-binaries)

The official binary releases of containerd are available for the `amd64` (also known as `x86_64`) and `arm64` (also known as `aarch64`) architectures.

Typically, you will have to install [runc](https://github.com/opencontainers/runc/releases) and [CNI plugins](https://github.com/containernetworking/plugins/releases) from their official sites too.

#### Step 1: Installing containerd

Download the `containerd-<VERSION>-<OS>-<ARCH>.tar.gz` archive from [https://github.com/containerd/containerd/releases](https://github.com/containerd/containerd/releases) , verify its sha256sum, and extract it under `/usr/local`:

$ tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
bin/
bin/containerd-shim-runc-v2
bin/containerd-shim
bin/ctr
bin/containerd-shim-runc-v1
bin/containerd
bin/containerd-stress

The `containerd` binary is built dynamically for glibc-based Linux distributions such as Ubuntu and Rocky Linux. This binary may not work on musl-based distributions such as Alpine Linux. Users of such distributions may have to install containerd from the source or a third party package.

> **FAQ**: For Kubernetes, do I need to download `cri-containerd-(cni-)<VERSION>-<OS-<ARCH>.tar.gz` too?
> 
> **Answer**: No.
> 
> As the Kubernetes CRI feature has been already included in `containerd-<VERSION>-<OS>-<ARCH>.tar.gz`, you do not need to download the `cri-containerd-....` archives to use CRI.
> 
> The `cri-containerd-...` archives are [deprecated](https://github.com/containerd/containerd/blob/main/RELEASES.md#deprecated-features), do not work on old Linux distributions, and will be removed in containerd 2.0.

##### systemd

If you intend to start containerd via systemd, you should also download the `containerd.service` unit file from [https://raw.githubusercontent.com/containerd/containerd/main/containerd.service](https://raw.githubusercontent.com/containerd/containerd/main/containerd.service) into `/usr/local/lib/systemd/system/containerd.service`, and run the following commands:

systemctl daemon-reload  
systemctl enable --now containerd

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-21-at-11.32.20 PM-1024x498.png)

#### Step 2: Installing runc[](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#step-2-installing-runc)

Download the `runc.<ARCH>` binary from [https://github.com/opencontainers/runc/releases](https://github.com/opencontainers/runc/releases) , verify its sha256sum, and install it as `/usr/local/sbin/runc`.

install -m 755 runc.amd64 /usr/local/sbin/runc

The binary is built statically and should work on any Linux distribution.

How fun! CURL does not want to download the "runc.amd64" file from GITHUB. I had to use WGET. Hrrmmm....

#### Step 3: Installing CNI plugins[](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#step-3-installing-cni-plugins)

Download the `cni-plugins-<OS>-<ARCH>-<VERSION>.tgz` archive from [https://github.com/containernetworking/plugins/releases](https://github.com/containernetworking/plugins/releases) , verify its sha256sum, and extract it under `/opt/cni/bin`:

$ mkdir -p /opt/cni/bin
$ tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
./
./macvlan
./static
./vlan
./portmap
./host-local
./vrf
./bridge
./tuning
./firewall
./host-device
./sbr
./loopback
./dhcp
./ptp
./ipvlan
./bandwidth

The binaries are built statically and should work on any Linux distribution.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-12.26.22 AM-1024x580.png)

**Lets see where we are at now!** XD

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-12.28.13 AM-1024x894.png)

The containerd error is now gone. Time to investigate some more errors. So much work to get this K8S Linux Container (LXC) operational!

Well, maybe this is the way to remove the error:  
[https://stackoverflow.com/questions/41453263/docker-networking-disabled-warning-ipv4-forwarding-is-disabled-networking-wil](https://stackoverflow.com/questions/41453263/docker-networking-disabled-warning-ipv4-forwarding-is-disabled-networking-wil)

```
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-12.47.32 AM-676x1024.png)

I removed the # from the net.ipv4.ip\_forward=1 and then ran a reboot on the LXC and I still have the same error.

The `cat <<EOF` syntax is very useful when working with multi-line text in Bash, eg. when assigning multi-line string to a shell variable, file or a pipe.

### Enable IPv4 packet forwarding[](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional)

[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)  
  
To manually enable IPv4 packet forwarding:

```
# sysctl params required by setup, params persist across reboots

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that `net.ipv4.ip_forward` is set to 1 with:

```
sysctl net.ipv4.ip_forward
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-1.19.48 AM-794x1024.png)

Not so sure all those Read-only ignores are for the best but the net.ipv4.ip\_forward = 1 did check out!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-1.21.15 AM-1024x578.png)

..... to be continued..... it's almost 2am. WHHHYYY?!!!

* * *

## Good info on kubeadm reset: But not in the timeline of last night. :)

> [https://stackoverflow.com/questions/54975986/preflight-errors-in-kubernetes-installation](https://stackoverflow.com/questions/54975986/preflight-errors-in-kubernetes-installation)
> 
> ![](https://wbb.afh.mybluehost.me/wp-content/uploads/2024/12/Screenshot-2024-12-22-at-11.43.46 AM-777x1024.png)
