---
title: "Let me try and get Wordpress live on a Windows Server."
description: ""
pubDate: "Dec 22 2024"
categories: ["Uncategorized"]
heroImage: "/images/posts/IISwindows.webp"
---
In my last [post](/blog/2024/kubernetes-in-a-proxmox-lxc-what-could-go-wrong-part1) I realized there are so many privilege requirements to explore with the K8S LXC containers. I decided to take a break and see about hosting the Wordpress site on Windows Server.

Configuring with "kubeadm" over manually provisioning each node is a learning experience for sure. Harder since it's in a LXC - lot of unknowns versus my RPi cluster. Going to take a break so I can tinker with Windows Server - sometimes a GUI is comforting, lol.

1 hour later... I forgot my Windows Server 2025 password. Had to perform a complete reinstall since I couldn't reset through the CMD prompt easily.

Well, the initial setup of Windows Server is like any other standard OS installation. The launch page is very welcoming and has the "Add roles and features" link. Clicking that takes you to the web server setup.

## Step 1: Installing IIS and required tools.
* Open Server Manager and select Manage Add Roles and Features.
* In the Add Roles and Features Wizard, select Role-based or feature-based installation and click Next.
* In the Roles section, select Web Server (IIS) and click Next.
* In the Role Services section, ensure the following services are selected:

![](/images/posts/2024/12/Screenshot-2024-12-22-at-12.59.32 PM.avif)

* Click Next, then Install.
* Verify the Installation by visiting localhost, you will see the following default page of IIS.

Opening the browser and typing in "localhost" seems to be functioning well. Internet was down though... attempted to install VirtIO drivers - didn't see it complete though. Server rebooted itself while I was on another Desktop. Network is functioning fine now.

![](/images/posts/2024/12/Screenshot-2024-12-22-at-1.03.37 PM.avif)

## This is getting out of hand quick! PAUSE for research.  
[https://www.tothenew.com/blog/setting-up-wordpress-website-on-windows-server-2022-with-iis/](https://www.tothenew.com/blog/setting-up-wordpress-website-on-windows-server-2022-with-iis/)

## Step 2: Install Microsoft Visual C++ Redistributable Version
* Download the latest version of Microsoft Visual C++ from [here](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170).
 
![](/images/posts/2024/12/visual-c++.avif)

* Applications that are built using Microsoft C and C++ tools require Visual C++.
* For WordPress, it is required by various dependencies, including PHP extensions, database servers, and web server components to function properly.
 
## Step 3: Install PHP
* Download the latest Non-Thread Safe PHP version from [here](https://windows.php.net/download).
* Extract the downloaded ZIP file to C:\\php
* Add C:\\PHP to the system’s PATH environment variable.
* Right-click This PC Properties Advanced system settings.
* Click Environment Variables, then find the path variable in the system variables section and click Edit.
* Click New and add C:\\php.

![](/images/posts/2024/12/php-variable.avif)


* Rename the php.ini-production file name to php.ini
 
## Step 4: Configure PHP for WordPress
* Open C:\\php\\php.ini file in a text editor.
* Uncomment and change the following values:

cgi.force\_redirect = 0
cgi.fix\_pathinfo = 1
fastcgi.impersonate = 1
fastcgi.logging = 0
extension=mysqli
extension=pdo\_mysql
 
## Step 5: Configure IIS for PHP
A) Handler Mapping:

* Open IIS Manager.
* Select your server in the left panel.
* Double-click Handler Mappings.
* Click Add Module Mapping in the Actions pane.
* Set the Request Path to \*.php, Module to FastCgiModule, Executable to C:\\php\\php-cgi.exe, and any Name to it.
 
![](/images/posts/2024/12/handler-mapping.avif)

B)  Default Document:
* Click on the Default Documents option.
* Add default.php and index.php
 
![](/images/posts/2024/12/default-document.avif)
 
## Step 6: Create and configure the IIS Application Pool
A) Creating a new Application Pool
* Right-click on the “Application Pools”
* Click on the  “Add Application Pool” window.
 
![](/images/posts/2024/12/application-pool.avif)
 
B) Configure the Application Pool
* In the “Application Pools” list, find the newly created application pool WordPress.
* Right-click on it and select “Set Application Pool Defaults”
* Click on the ApplicationPoolIdentity Custom Account Set credentials Provide the Username and Password of the Administrator.

![](/images/posts/2024/12/applicaiton-pool-user.avif)

C) Change Application Pool for Default Web Site

* Click on Default Web Site.
* Right-click on the Basic Setting under Edit Site at the extreme right.
* Select the Application Pool that we created earlier i.e. WordPress.

![](/images/posts/2024/12/default-application-pool.avif)

## Step 7: Install MySQL - WHERE THINGS GET MURKY!!!!
* Download the MySQL Installer from [here](https://dev.mysql.com/downloads/installer/).
* Run the installer and choose the Server and Workbench option.
* Follow the prompts to complete the installation, and set a root password when prompted.
* Once installed, open the MySQL Workbench create a database for WordPress, and grant all privileges to the root user.

## Note: 
You can use the mysql cli commands to create the database. You can create a new user for WordPress, however I will be using the root user only.

So major jump off course here: Trying to understand MySQL from the Windows perspective. Luckily, running through BASH on my first install to the Ubuntu Server I have some familiarity with what's happening - it's just in a GUI now. XD  
[https://www.michaelstults.com/2014/10/how-to-setup-mysql-workbench-database-for-wordpress-on-windows-server/](https://www.michaelstults.com/2014/10/how-to-setup-mysql-workbench-database-for-wordpress-on-windows-server/)  
I'll come back to this - since I have already setup the database.

## Step 8:  Download and Configure WordPress
* Download the latest version of WordPress from [here](https://wordpress.org/download/).
* Extract the ZIP file to C:\\inetpub\\wwwroot\\wordpress.
* Rename the wp-config-sample.php to wp-config.php
* Open the wp-config.php file in a text editor and provide the database details.

![](/images/posts/2024/12/database-details.avif)

Emphasis on the DB\_NAME, DB\_USER, and DB\_PASSWORD. As we have learned in the past with Linux - if one thing is off in the config, you will not be able to authenticate to your database created in MySQL.

The Windows Server is function as expected. Wordpress is available on the localhost - however pushing it to the Internet may not work the way I expected. I'm going to investigate linking the local site to a domain through a Cloudfare Tunnel.

Time to look at purchasing a domain!
