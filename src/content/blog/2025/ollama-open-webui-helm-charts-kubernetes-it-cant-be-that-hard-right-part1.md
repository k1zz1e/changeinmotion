---
title: "Ollama, open-webui, helm charts &amp; kubernetes - it can't be that hard right? #Part1"
description: ""
pubDate: "Nov 5 2025"
categories: ["AI", "helm", "Kubernetes", "Linux", "ollama", "Talos"]
tags: ["helm", "helm charts", "kubectl", "Kubernetes", "ollama", "open-webui", "PVC", "synology-csi", "Talos"]
heroImage: "/images/posts/ollamahelmpart1.webp"
---
So my previous posts have went from Talos OS installation across my cluster to the configuration of K8S and Helm. Now it's time to start deploying some applications and services. Why not start with Ollama? I mean, how bad can it be with a CPU only based cluster - 2 tokens per second? Let's hope for more or I may have to add more nodes.

Moving on! [Artifact Hub page](https://artifacthub.io/packages/helm/ollama-helm/ollama) for the ollama helm chart. You can see below installation is fairly simple with helm.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/ollama-helm-site-1024x954.png)

Run a few magic commands and presto!

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/ollama-helm-chart-install-1024x433.png)

Now let's see whats happening under the hood. Initially my instincts fired off:

kubectl get pods --all-namespaces

But then I got smart:

kubectl get all -n ollama

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/kubectl-get-all-ollama.png)

So as you can see the ollama pod, service, deployment, and replicaset are all up and running. Of course now, we have to interact with our model. And to do that we need to utlize the AI standard of standards [Open WebUI](https://openwebui.com/).

#Fun tip - you can log in to pods with a shell via this command:  
kubectl exec -it -n ollama pod/ollama-75d5ccbd95-jq7wr -- /bin/sh

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Artifact-HUB-OpenWEBUI-1024x926.png)

Another fairly straight forward deployment with helm charts. After the install is completed you will land on the default screen in the bash terminal.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/OPEN-WEBUI-Welcome-1024x581.png)

A went ahead and loaded up the address in the browser: 127.0.0.1:8080 - and to my surprise, I was unable to connect. It's never this simple when there are configurations involved.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Firefox-Unable-to-Connect-1024x412.png)

Sometimes deployments can take a minute depending on hardware but always best to investigate.

```
kubectl get pods --all-namespaces
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Open-WebUI-pod-failure-1024x446.png)

As seen above two pods have failed to start:  
  
open-webui-0  
open-webui-pipelines-755455d644-cvhwz

### Welcome to DevOPs!  
It's time to start digging.

Since we already know the name of the two pods that have failed to run we need to see what is going on with them specifically:

kubectl describe pod open-webui-0  
kubectl describe pod open-webui-pipelines-755455d644-cvhwz

Both return similar "Event" warnings at the bottom. The screenshot below was taken after removing the NFS storage-class.yml, so the error warning is different.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/kubectl-describe-open-webui-1024x663.png)

My specific error output:

> Failed to login with target iqn \[iqn.2000-01.com.synology:DS1520.pvc-998cca64-aeda-472e-95a7-5a36b09b74a2\], err: chroot: can't execute '/usr/bin/env': No such file or directory (exit status 127)

If you notice in the output "synology:DS1520" that is because I am using Synology's SAN Manager application as an iSCSI target:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Synology-SAN-Manager-1024x792.png)

Nodes in my cluster can request and use storage on the NAS to setup persistent volumes. I chose the [synology-csi](https://github.com/SynologyOpenSource/synology-csi) installation from one of the options on Sidero Labs storage [page](https://docs.siderolabs.com/kubernetes-guides/csi/synology-csi) for Talos OS. Additionally, have previous experience with CEPH in Proxmox, so I wanted to try something else.

* * *

So initially I believed this to be an error with a configuration file I modified and forgot about in the local synology-csi files. The storage-class.yml was modified to the NFS protocol prior to executing the synology-csi [deploy.sh](https://github.com/SynologyOpenSource/synology-csi/blob/main/scripts/deploy.sh) script.

Below you can see the templates for the [synology-iscsi](https://github.com/SynologyOpenSource/synology-csi/tree/main) storage installation:

**NFS Protocol:**

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: synostorage-nfs
provisioner: csi.san.synology.com
parameters:
  protocol: "nfs"
  dsm: "192.168.1.1"
  location: '/volume1'
  mountPermissions: '0755'
mountOptions:
  - nfsvers=4.1
reclaimPolicy: Delete
allowVolumeExpansion: true
```

**iSCSI Protocol**:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
  name: synostorage
provisioner: csi.san.synology.com
parameters:
  fsType: 'btrfs'
  dsm: '192.168.1.1'
  location: '/volume1'
  formatOptions: '--nodiscard'
reclaimPolicy: Retain
allowVolumeExpansion: true
```

I initially thought I needed to configure it for NFS. But as I am going through the documentation I am getting conflicting information.

So I uninstalled the synology-csi with the included script and restored the storage-class.yml file to the iSCSI Protocol. You can actually see at the bottom of the script where it show synostorage-nfs during the uninstall.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Synology-csi-Uninstall-Script-1024x514.png)

Now let's reinstall synology-csi:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/Deploy-synology-csi-1024x621.png)

And then check to see how are pods, PVCs, and storage are running:

kubectl get pods -n synology-csi  
kubectl get storageclass  
kubectl get pvc

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/11/synology-csi-install-check-1024x242.png)

So if you look above you can see open-webui is configured correctly this time to STORAGECLASS synology-iscsi but still does not run correctly. I am getting a few different errors and I believe this is an issue specifically with PVCs and my helm charts.

##### Persistent Volume Claims Notes:

> When persistence is enabled in the Helm chart's `values.yaml` file (`persistence: enabled: true`), the chart creates a `PersistentVolumeClaim` (PVC). This PVC dynamically requests storage from the **default `StorageClass` defined within your specific Kubernetes cluster**. 

As seen in the quote above, if persistence is enabled, the PVC will request storage from the default storageclass in my cluster. I am going to have to move on to part II tomorrow. I need to review the configurations in full for synology-csi, ollama and Open WebUI.

> "It's interesting to me that my Grafana stack ran effortlessly after a helm chart install - but the others are a bit trickier." ~ Kyle's brain

* * *

The conclusion of all my silly mistakes here has led me to realize I need to integrate GitHub into my work and configure it with Visual Studio Code. I don't know why I waited so long.

#### [https://github.com/k1zz1e](https://github.com/k1zz1e)
