---
title: "Windows Server 2022 Active Directory Lab Setup"
description: ""
pubDate: "Jan 29 2025"
categories: ["Uncategorized"]
heroImage: "/images/posts/windowsserverAD.webp"
---
Well, this has been coming for a while now. I just got really distracted with fun stuff - an Ethical Hacking course + studying for my Security+ certificate. I've put off acquiring certs for far to long! They expire, so I've had to be strategic. Back on task!

My Windows Server 2025 was running a Wordpress site, Sonarr, Radarr, and Jackett but decided I to transition the media services to a headless Ubuntu server to save on resources. I thought I was going to use this VM for an Active Directory lab, but it seemed better to use Windows Server 2022. I feel many companies are still running Window Server 2019/2022 - maybe even older depending on the companies budget.

_The configuration:_  
**Windows Server 2022: 192.168.3.2  
Windows 10: 192.168.3.3**  
**Domain: homelab.local**

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-7.07.10 PM-1024x774.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-7.20.04 PM-1024x774.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-7.19.49 PM-1024x774.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-7.33.16 PM-1024x774.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-7.29.38 PM-1024x774.png)

The Active Directory Pro [website](https://activedirectorypro.com/create-active-directory-test-environment/) was a tremendous help in this process. After the initial setup and configuration of the static IP/DNS I changed up the naming scheme on the scripts for rolling out the Active Directory information based on data from CSV files.

[AD Homelab Scripts](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/AD-Homelab-Scripts.zip)[Download](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/AD-Homelab-Scripts.zip)

I then had to step out to my UDM Pro console to configure the domain "homelab.local" and configure a new VLAN for this project. Here is a good writeup at [LazyAdmin](https://lazyadmin.nl/home-network/unifi-local-and-server-dns-settings/) for understanding the UniFi control plane setup of your domain and fixed IP address.

After that I jumped in to Window Server 2022 and ran the create\_ous.ps1 file in the PowerShell ISE. You may have to open it independently - Windows initially opened my PS1 file in Notepad.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-9.02.10 PM-1024x771.png)

I then continued on with running "create\_groups" and "create\_users" - which took a little bit longer due to the amount of users. Lastly I went in and ensured I had my name setup in the IT Homelab User section.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.29.19 PM-1024x771.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.26.13 PM-1024x771.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.29.46 PM-1024x771.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.33.33 PM-1024x771.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.39.42 PM-1024x771.png)

Now that Windows Server 2022 is configured - I need to fire up the Windows 10 VM in my cluster and configure it to the server. Luckily, I already have a Windows 10/11 VM on standby! The Windows 10 configuration - individual IP and then point at your Active Directory server for the "Preferred DNS server".

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.43.58 PM-1024x771.png)

You may have to restart between here - once you return, change the "Member of Domain" to reflect homelab.local. This screenshot is actually incorrect but you can see above the Workgroup box - the correct one to fill in.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-10.47.34 PM-1-1024x771.png)

Things after this are as I expected. Utilizing my login I created in the Active Directory server earlier I was able to login to the Windows 10 VM.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-11.11.11 PM-1024x771.png)

And then prompted to change my password in accordance with rules configured under my user account.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/01/Screenshot-2025-01-28-at-11.11.20 PM-1024x771.png)

Now that I am logged in I have confirmed everything is working as intended. So with these two VMs configured I will be able to free explore the administration and configuration of Active Directory, in depth usage of PowerShell and scripting in a Windows environment.

Interestingly enough I got an itch today to tinker with Ansible - I want to try and execute a playbook to configure my Raspberry Pi cluster. I guess I will see how I feel about it tomorrow. Still a lot of Ethical Hacking to do this week!
