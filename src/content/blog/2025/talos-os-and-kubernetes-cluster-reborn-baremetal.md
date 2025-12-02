---
title: "Talos OS and Kubernetes.  Cluster reborn! #baremetal"
description: ""
pubDate: "Oct 23 2025"
categories: ["Kubernetes", "Linux", "Talos"]
tags: ["K8S", "Kubernetes", "Linux", "OS", "Talos"]
heroImage: "/images/posts/sidero.jpg"
---
For starters here, lets define my equipment for this project.

*   3 x HP Elitedesk mini PCs (1 control plane, 2 worker nodes)
*   A separate monitor, keyboard and mouse to monitor node output
*   A MacBook laptop to remote in via the Talos API

I chose a bare metal install here to avoid virtualization. Sometimes it's nice to avoid additional layers of abstraction and get your hands on the hardware. But if you really want to, I'm sure you can virtualize all this or use Docker.

The starting focus will be on configuring the control plane. This is the primary node in Kubernetes that will issue instructions to our cluster of worker nodes.  
  
So I am skipping the simple standard stuff here; plug in everything, ping tests, configuration of your router, VLANs, etc. You should know your way around those things before you ever get to Kubernetes.

## ./begintalos.sh #seriously

The beginning of all things starts with documentation: [https://docs.siderolabs.com/talos/v1.8/overview/what-is-talos](https://docs.siderolabs.com/talos/v1.8/overview/what-is-talos)

I start below by generating three configuration files, naming my cluster and providing the location of my control node IP: 192.168.3.2. Workers are added later.

```
talosctl gen config TalosK8S https://192.168.3.2:6443
```

Output in my Warp terminal can be seen below and then corresponding mess because I forgot to make a folder for the output.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Warp-Generate-Talon-Configs-1024x640.png)

So make a folder for your configuration files and then you can open the three config YAML files from our control node in Visual Studio Code for editing.

## Three config YAMLs defined

#### controlplane.yaml

*   Defines the configuration for the Talos control plane node.
*   Contains settings for networking, storage, and Kubernetes API components.
*   Manages and coordinates the entire Kubernetes cluster, including scheduling and control services.

### worker.yaml

*   Provides configuration for worker nodes that run application workloads.
*   Includes machine-specific settings such as IP addresses, interfaces, and disk details.
*   Allows the node to join and operate under the control plane’s management.

### talosconfig

*   A client-side configuration file used by the talosctl command-line tool.
*   Stores cluster access credentials, endpoints, and authentication information.
*   Enables secure remote management and administration of Talos nodes.

You can see below in the left column I have created a couple of different files. The Customized folder for making lots of mistakes with my configurations and the Default folder - because you always need a backup.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Default-controlplane-yaml-1007x1024.png)

So if you are anything like me, you like to edit a lot of things and make tons of errors. So there was a bit of delay in applying my configuration to the cluster. But I came across the solution:

In the documentation you can see the very same error I encountered by not specifying the target disk (one of many errors but this was the final boss).

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Modifying-the-Machine-Config-841x1024.png)

[https://docs.siderolabs.com/talos/v1.8/getting-started/getting-started](https://docs.siderolabs.com/talos/v1.8/getting-started/getting-started)

I changed a few lines in **controlplane.yaml** and **worker.yaml** related to the installation drive as seen above and that fixed my issues applying the configuration.

_NOTE: Ensure the talos version match on the client and node or you may get stuck here. Likely will only happen if you are doing a reinstall_ of the cluster.

The two lines:

```
debug: true # Enable verbose logging to the console.
```

```
install:
      disk: /dev/nvme0n1 # The disk used for installations.
```

To see the disks on your node, use the command as shown in the documentation:

```
talosctl get disks -i -n 192.168.3.2
```

You can see the results of this below:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/talosctl-get-disks-1024x115.png)

Our YAML file by default when being generated has the disk set to **/dev/sda**. But the correct disk is **/dev/nvme0n1** as seen above. So edit your **_controlplane.yaml_** and change the drive string to reflect the correct drive; it's also your installation drive for Talos.

Once you are done with your updates to the YAML config files. Apply the configuration file _**controlplane.yaml**_ by using the following command:

```
talosctl apply-config --insecure --nodes 192.168.3.2 --file controlplane.yaml
```

That will apply the config which you can visually see happening on the monitor for the node (if you setup a bare metal machine). Then after that is complete, you can bootstrap the control node with the following:

_NOTE: Your node will be stuck in a_ "_booting" status_ - until you bootstrap. You can see this visually on the talos dashboard

```
#remove old client locally (your pc)
sudo rm /usr/local/bin/talosctl

#redownload talos
curl -sL https://talos.dev/install | sh
```

* * *

```
talosctl bootstrap -n 192.168.3.2 -e 192.168.3.2 --talosconfig talosconfig
```

After that you should see your control node show "Running" under the "STAGE" section.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Talos-Health-1024x120.png)

If you go to your browser now and type in [https://192.168.3.2:6443](https://192.168.3.2:6443) you can see the control node is receiving connections and providing output:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Control-plane-in-the-browser.png)

You can pull up the health dashboard to see what is happening on the node with the following command:

```
talosctl --nodes 192.168.3.2 --endpoints 192.168.3.2 dashboard --talosconfig=./talosconfig
```

This is the same output you would see on a monitor connected to the node. There is a couple of views you can change to at the bottom with F2 and F3. One is similar to [HTOP](https://htop.dev/). If you don't know about how cool that is, you are missing out - it's so RAD!

Oh, here is the health dashboard:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Health-Dashboard-1024x676.png)

[https://docs.siderolabs.com/talos/v1.9/deploy-and-manage-workloads/interactive-dashboard](https://docs.siderolabs.com/talos/v1.9/deploy-and-manage-workloads/interactive-dashboard)

And additionally, you can merge your new cluster (of one node lol) with _kubectl_ in your local Kubernetes configuration on the computer you are using to access the cluster. The command:

```
talosctl kubeconfig alternative-kubeconfig.yaml --nodes 192.168.3.2 --endpoints 192.168.3.2 --talosconfig=./talosconfig
```

Run "**ls"** and you will see the alternative-kubeconfig.yaml file now in the current directory. You may need to run another command if you didn't use **brew** to install.

#optional requirement to run kubectl locally  
export KUBECONFIG=./alternative-kubeconfig.yaml

You may be asking... what does this mean? Well, if you haven't already explored Kubernetes this could get a little weird. A good place to read as always is the documentation: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)  
[](https://kubernetes.io/docs/tasks/tools/#kubectl)

> **kubectl**  
>   
> The Kubernetes command-line tool, [kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/), allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.   

So if you haven't already you will need to install kubectl. See here: [https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos)

I used homebrew on MacOS so my command is shown as the following:

```
brew install kubectl
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Homebrew-kubectl-Install-1024x373.png)

Then to test if everything is working we run a simple command on my host computer (MacBook):

kubectl get nodes

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/kubectl-get-nodes-1024x116.png)

Quite exciting to see a functioning node! And this time, no SD cards or Raspberry Pi cluster shenanigans. I did enjoy my RPi cluster projects greatly – if you haven't started anywhere, definitely start there to get a feel for K3S or MicroK8S: [https://k3s.io/](https://k3s.io/) and [https://microk8s.io/](https://microk8s.io/). Both are lightweight versions of Kubernetes, and MicroK8S is baked into Ubuntu, which functions rather smoothly.

And if you got a 3D printer, maybe get a little wild and print a Pi cluster rack.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Pi-Cluster-3D-Print-1024x567.png)

> **break; _//_** _DAY TWO BEGINS_

### **Worker nodes!**

Firing up node two with the Talos flash drive. Everything checks out as expected. IP address grabs effortlessly with the static configuration in my router. Going through the first control plane node setup really makes things smoother for the next ones!

```
talosctl get disks -i -n 192.168.3.3
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Worker-Node-Disks-1024x67.png)

Change the configuration of your worker.yaml in Visual Studio Code (or Kate, or any syntax friendly text editor!) to incorporate the correct disk title: /dev/nvme0n1

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Worker-YAML-1024x1021.png)

Interesting error encountered! I tried to reach the dashboard on the new worker node and was unable to reach it with different variations of the the talos dashboard command. Seems you cannot access another node with configuration files generated by another. I suppose I will go forward and apply the worker.yaml file.

talosctl apply-config --insecure --nodes 192.168.3.3 --file worker.yaml

If you have setup an physical monitor you can view the process changes across the various status monitors on the [dashboard](https://docs.siderolabs.com/talos/v1.9/deploy-and-manage-workloads/interactive-dashboard). I am mainly focusing on STAGE.

#### STAGE: Installing

_Talos is installing from the flash drive...._

#### STAGE: Booting

_Talos is booting off the installation disk of your node...._

#### STAGE: Running

_Talos is live on the installation disk of your node... pull flash drive..._

The difference can be seen at the top of the dashboard under CLUSTER (two machines). I am logged in to the control plane node here:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/Talos-Dashboard-Two-Machines-1024x138.png)

Quite simple to continue adding nodes now for Kubernetes to run wild on. I have three set aside for this cluster. The third one is as simple as the previous.

*   Check disk for drive name
*   Change worker.yaml accordingly
*   Apply config to IP address of third node, fourth, fifth, etc

The config file generated by the control plane node has certification authority keys and certificate variables in it that allow for the worker nodes to join the cluster and communicate with each other.

Adding third node.... queue elevator music.....

Before the finish line lets edit our talosconfig file with the nodes and endpoints.

export TALOSCONFIG=/home/USER?/talos/talosconfig  
  
talosctl config endpoint 192.168.3.2  
  
talosctl config nodes 192.168.3.3 192.168.3.4

That will add the endpoints/nodes to your talosconfig file to ensure it know the target of talosctl commands.

Now lets see if we have them all configured and in our cluster.

```
kubectl get nodes
```

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/10/kubectl-get-nodes-1-1024x583.png)

_Oh My Zsh?! Look at that cute terminal!_

Decided to get clever before that last command and renamed my user@device in the zsh terminal. I named it with the following command:

PS1='%~ '

That essentially blanks out the terminal - then you wont see your user name and device information before commands. You can change it to anything you want though: _PS1='%~ TalosMSTR"_ :)

So now that we have a functioning cluster on [Talos Linux](https://www.talos.dev/) we can roll into any of the typical fun you can imagine with a Kubernetes cluster. I'll get more into that on the next post.
