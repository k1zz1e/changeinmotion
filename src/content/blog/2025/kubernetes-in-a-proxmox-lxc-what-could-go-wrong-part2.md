---
title: "Kubernetes in a Proxmox LXC - what could go wrong? #part2"
description: ""
pubDate: "Jan 3 2025"
categories: ["Uncategorized"]
heroImage: "/images/posts/proxmoxk8s.webp"
---
Beginning again with the great work left off on from the previous [posts](/blog/2024/kubernetes-in-a-proxmox-lxc-what-could-go-wrong-part1) - I have started up fixing errors.

![](/images/posts/2025/01/Screenshot-2025-01-02-at-2.54.13 PM.avif)

Two things here - kubeadm is trying to reach out to the Internet. Not sure if that is going to cause any complications later. For the time being I am going to to disregard that error.

```
kubectl version 

Client Version: v1.32.0
Kustomize Version: v5.5.0
```

```
kubeadm ini --kubernetes-version v1.32.0
```

![](/images/posts/2025/01/Screenshot-2025-01-02-at-2.58.19 PM.avif)

Appears the kubeadm wants to generate new manifests. I simply moved the ones located in /etc/kubernetes/manifests to a backup folder just in case.

```
#THIS IS THE MANUAL WAY
mkdir /etc/kubernetes/manifests/backup
mv -t /etc/kubernetes/manifests/backup etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml
```

```
#PREFERRED WAY
kubeadm reset
```

![](/images/posts/2025/01/Screenshot-2025-01-02-at-3.08.15 PM.avif)

Seems to be a healthy initiation. Until... it wasn't.

![](/images/posts/2025/01/Screenshot-2025-01-02-at-3.17.28 PM.avif)

the kubelet-check hit an error - attempting to allow port access on 10248 to see if that fixes the issue.

```
ufw allow 10248
```

Still nada!

Removed the config.toml file from /etc/containerd and then ran

```
systemctl restart containerd && systemctl restart kubelet
```

```
systemctl status containerd  && systemctl status kubelet
```

Both appear to be running fine now.

![](/images/posts/2025/01/Screenshot-2025-01-02-at-5.47.42 PM.avif)

## **DISTRACTION!**

Just out of curiosity I am going to see how this works on a standard Ubuntu Server install - I feel like there is either missing dependencies or a change with Kubernetes v1.32 and container runtimes that I am not tracking yet. I've read a few things about a transition away from the pre-installed [docker engine](https://kubernetes.io/docs/setup/production-environment/container-runtimes/). Plus a break would be nice!

Since I already had a working Ubuntu Server on node1 I went ahead and cloned it to node3. Later I will attempt to make another template for Ubuntu installs - something was off on the last one. I was surprised how easily Proxmox cloned and sent the OS to the other node. Just need to change the hostname now and should be good to go.

```
hostnamectl
```

![](/images/posts/2025/01/Screenshot-2025-01-02-at-8.54.14 PM.avif)

Looks like the Machine IDs are identical curious how that will conflict with anything after changing the hostname. It can be edited here: /etc/machine-id.

UPDATE: The Machine ID determines DHCPs IP address issued to the VM. I changed the first two numbers from 72 to 69, then reboot. DHCP then issued a unique IP address to the VM.

```
hostnamectl hostname lama2
```

Then change from old hostname to new hostname by editing the hosts file with nano:

![](/images/posts/2025/01/Screenshot-2025-01-02-at-9.05.51 PM.avif)

I don't believe cloud-init was applicable to this install. I believe I went through manual installer a while back. I am going to change it anyway on recommendation of information I was reading on [Linuxize](https://linuxize.com/post/how-to-change-hostname-on-ubuntu-22-04/).

![](/images/posts/2025/01/Screenshot-2025-01-02-at-9.09.08 PM.avif)

```
sudo nano /etc/cloud/cloud.cfg
```

![](/images/posts/2025/01/Screenshot-2025-01-02-at-9.07.26 PM.avif)

> Making a backup before doing any major changes to my VMs is my new favorite thing! It's simply a lifesaver!

Proxmox backup of the VM to my Synology server. I need more storage space. I am going to get really backup happy.

![](/images/posts/2025/01/Screenshot-2025-01-02-at-9.19.07 PM.avif)

After a brief delay, I reconfigured the Machine-ID for the VM to be issued a new IP address from DHCP. Perhaps I need to get that template setup sooner. :D

Time for sleep! Until tomorrow!
