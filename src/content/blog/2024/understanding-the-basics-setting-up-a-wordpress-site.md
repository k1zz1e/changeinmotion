---
title: "Understanding the basics - setting up a Wordpress site."
description: ""
pubDate: "Dec 13 2024"
categories: ["Wordpress"]
tags: ["apache2", "mysql", "spice", "ubuntu server", "vnc", "wordpress"]
heroImage: "/images/posts/wordpress.webp"
---
Ramblings and adventures in the backend of website hosting through MySQL, Apache2, Wordpress and a lot of Linux navigation.

* * *

Lets begin setting up a Wordpress site on my Ubuntu Server. Mostly used the guide found on:

## 2\. Install Dependencies

To install PHP and Apache, use following command:

```
sudo apt update
sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
                 php-bcmath \
                 php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip
```

## 3\. Install WordPress

We will use the release from Wordpress rather than the APT package in the Ubuntu Archive, because this is the preferred method from upstream WordPress. This will also have fewer “gotcha” problems that the WordPress support volunteers will not be able to anticipate and therefore be unable to help with.

Create the installation directory and download the file from Wordpress

```
sudo mkdir -p /srv/www
sudo chown www-data: /srv/www

curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```

Note that this sets the ownership to the user `www-data`, which is potentially insecure, such as when your server hosts multiple sites with different maintainers. You should investigate using a user per website in such scenarios and make the files readable and writable to only those users. This will require configuring PHP-FPM to launch a separate instance per site each running as the site’s user account. In such setup the `wp-config.php` should (read: if you do it differently you need a good reason) be readonly to the site owner and group and other permissions set to no-access (`chmod 400`). This is beyond the scope of this guide, however.

## 4\. Configure Apache for WordPress

Create Apache site for WordPress. Create /etc/apache2/sites-available/wordpress.conf with following lines

```
<VirtualHost *:80>
    DocumentRoot /srv/www/wordpress
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo
        DirectoryIndex index.php
        Require all granted
    </Directory>
    <Directory /srv/www/wordpress/wp-content>
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```

Enable the site with:

```
sudo a2ensite wordpress
```

Enable URL rewriting with:

```
sudo a2enmod rewrite
```

* * *

**#START** **DISTRACTION** **:)**

Taking a moment to deconflict the copy/paste function with the VNC console in Proxmox – NoVNC does not allow easy copy/paste function for scripts. Will investigate SPICE:  
https://pve.proxmox.com/wiki/SPICE#Requirements\_for\_SPICE  
  
To enable it set the Display in the Hardware section of the Proxmox VM to Spice

```
Shutdown -r now
Restart VM in Proxmox
```

SPICE is not compatible with MacOS. Alternative is SSH natively in the Ubuntu Server or utilize Tampermonkey script to copy/paste in NoVNC  
https://gist.github.com/amunchet/4cfaf0274f3d238946f9f8f94fa9ee02#file-novnccopypasteproxmox-user-js

**Ubuntu server missing nano!**

```
sudo apt install nano
```

```
cd /etc/apaches/sites-available
nano wordpress.conf
```

**Tampermonkey is finicky – I will SSH into Ubuntu Server natively**

```
Sudo nano /etc/apache2/sites-available/wordpress.conf
```

Save/close nano

**#END **DISTRACTION****

Back to guide [https://ubuntu.com/tutorials/install-and-configure-wordpress#1-overview](https://ubuntu.com/tutorials/install-and-configure-wordpress#1-overview)  
  
Configure database

```
Sudo mysql -u root
```

```
mysql> CREATE DATABASE wordpress;
Query OK, 1 row affected (0,00 sec)
```

```
mysql> CREATE USER wordpress@localhost IDENTIFIED BY '<your-password>';
Query OK, 1 row affected (0,00 sec)
```

```
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
 -> ON wordpress.*
 -> TO wordpress@localhost;
Query OK, 1 row affected (0,00 sec)
```

```
mysql> FLUSH PRIVILEGES;
Query OK, 1 row affected (0,00 sec)
```

```
mysql> quit
```

## Configure WordPress to with the database

Copy the sample configuration file to wp-config.php:

```
sudo -u www-data cp /home1/wbbafhmy/public_html/wp-config-sample.php /home1/wbbafhmy/public_html/wp-config.php
```

Next, set the database credentials in the configuration file (do not replace database\_name\_here or username\_here in the commands below. Do replace <your-password> with your database password.):

```
sudo -u www-data sed -i 's/database_name_here/wordpress/' /home1/wbbafhmy/public_html/wp-config.php
```

```
sudo -u www-data sed -i 's/username_here/wordpress/' /home1/wbbafhmy/public_html/wp-config.php
```

sudo -u www-data sed -i 's/password\_here/<PASSWORD>/' /home1/wbbafhmy/public\_html/wp-config.php

Finally, in a terminal session open the configuration file in nano:

```
sudo -u www-data nano /home1/wbbafhmy/public_html/wp-config.php
```

## Find the following to configure with Salt:

Delete those lines (ctrl+k will delete a line each time you press the sequence). Then replace with the content of https://api.wordpress.org/secret-key/1.1/salt/. (This address is a randomiser that returns completely random keys each time it is opened.) This step is important to ensure that your site is not vulnerable to “known secrets” attacks. Save and close the configuration file by typing ctrl+x followed by y then enter

## Configure WordPress

Unable to login at http://localhost/ in browser – Ubuntu Server is a headless install on Node1.

Switched over to Kali linux VM on Node3 – nmap revealed port 80 is hosting an Apache server on Node1. Utilized Firefox and sent request https://wbb.afh.mybluehost.me and received a page reflecting an error. Was not anticipating this…

Config must be done manually. Solution is using the WP-CLI: https://wp-cli.org/#installing

## Installing

Downloading the Phar file is our recommended installation method for most users. Should you need, see also our documentation on alternative installation methods (Composer, Homebrew, Docker).

Before installing WP-CLI, please make sure your environment meets the minimum requirements: UNIX-like environment; limited support in Windows environment PHP 5.6 or later WordPress 3.7 or later. Versions older than the latest WordPress release may have degraded functionality.  Once you’ve verified requirements, download the wp-cli.phar file using wget or curl:

```
"curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar"
```

Next, check the Phar file to verify that it’s working:

```
php wp-cli.phar --info
```

To use WP-CLI from the command line by typing wp, make the file executable and move it to somewhere in your PATH. For example:

```
chmod +x wp-cli.phar
```

```
sudo mv wp-cli.phar /usr/local/bin/wp
```

If WP-CLI was installed successfully, you should see something like this when you run wp --info:

```
wp –info
```

OS: Linux 5.10.60.1-microsoft-standard-WSL2 #1 SMP Wed Aug 25 23:20:18 UTC 2021 x86\_64  
Shell: /usr/bin/zsh  
PHP binary: /usr/bin/php8.1  
PHP version: 8.1.0  
php.ini used: /etc/php/8.1/cli/php.ini  
MySQL binary: /usr/bin/mysql  
MySQL version: mysql Ver 8.0.27-0ubuntu0.20.04.1 for Linux on x86\_64 ((Ubuntu))  
SQL modes:  
WP-CLI root dir: /home/wp-cli/  
WP-CLI vendor dir: /home/wp-cli/vendor  
WP\_CLI phar path:  
WP-CLI packages dir: /home/wp-cli/.wp-cli/packages/  
WP-CLI global config:  
WP-CLI project config: /home/wp-cli/wp-cli.yml  
WP-CLI version: 2.11.0

# Updating:

You can update WP-CLI with wp cli update (doc), or by repeating the installation steps. If WP-CLI is owned by root or another system user, you’ll need to run sudo wp cli update.

**\---- Not familiar with the WP-CLI yet -----**

Possible issue with database setup

```
sudo mysql -u root
```

```
DROP DATABASE wordpress;
```

Now attempting to recreate database with guide:

```
mysql> CREATE USER wordpress@localhost IDENTIFIED BY '<password>';
```

```
ERROR 1396 (HY000): Operation CREATE USER failed for 'wordpress'@'localhost'
```

Have to drop each the DATABASE and the USER

```
DROP USER wordpress@localhost
Query OK, 0 rows affected (0.01 sec)
```

Possible error in the initial setup with:

```
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
-> ON wordpress.*
-> TO wordpress@localhost;
```

**\--- NOPE —STILL NOT CONNECTING TO DATABASE** **\---**

Went back to check Apache2 here: /etc/apache2/sites-available

Disabled wordpress.conf by renaming with mv command to wordpress.bk

Apache2 “It Works” site is accessible from Node3 Kali distro browser

So we don’t have an issue with access to the Ubuntu Server on Node1 and to my surprise it is accessible from my Macbook on the primary LAN 192.168.1.XXX. The Proxmox cluster is on 192.168.2.XXX

Re-enabled wordpress.conf by renaming back. Macbook reveals the “Error establishing a database connection. Will now abandon using Node3 Kali distro – was under the impression the VLAN would not be accessible from 192.168.1.XXX.

NOTE EXCERPT FROM: https://askubuntu.com/questions/1229521/apache-gives-it-works-page-instead-of-home-page-of-site

You will have to disable the example site and enable your moodle site. You can do this by using the a2ensite and a2dissite commands. To enable or disable a site hosted with Apache, you can use the 'a2ensite' and 'a2dissite' commands, respectively. Both commands use essentially the same syntax:

```
a2ensite [site]
a2dissite [site]
```

Where \[site\] is the name of your site's Virtual Host configuration file, located in /etc/apache2/sites-available/, minus the '.conf' extension. For example, if your site's Virtual Host configuration file is called moodle.com.conf and the "It Works!" configuration file is called example.com.conf, then the commands would look like:

```
a2ensite moodle.com
a2dissite example.com
```

After that, you can restart Apache by typing sudo systemctl restart apache2 and it should work now.

**NOTE FOR MYSQL permissions:**

```
mysql> GRANT ALL ON
```

or separately

mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER

Error discovered!!!

/home1/wbbafhmy/public\_html/wp-config.php had the incorrect password for the MySQL database – originally I caught the arrows on the end: ‘<helpme>’

Additionally, I had changed the password in MySQL when I dropped DATABASE and USER previously. They must sync as follows:

Under the wp-config.php in the nano editor:

```
/** Database password */
define( 'DB_PASSWORD', 'helpme' );
```

and under mysql:

```
mysql> CREATE USER wordpress@localhost IDENTIFIED BY '<helpme>';
```

## IT LIVES!
