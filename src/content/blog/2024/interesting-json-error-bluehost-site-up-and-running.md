---
title: "Interesting JSON error - Bluehost site up and running."
description: ""
pubDate: "Dec 30 2024"
categories: ["Wordpress"]
tags: ["cPanel", "curl", "JAVA", "JSON", "wordpress"]
heroImage: "/images/posts/notes2.webp"
---
The site is up and running without issue... until now. I started editing my very first post on the site due to bad formatting. I didn't really understand Wordpress yet, so it definitely needed to be polished up. Somehow in the process I discovered a error on Wordpress. And even though it's not a homelab project, it's a problem - and it must be vanquished!

![](/public/images/posts/2024/12/Screenshot-2024-12-26-at-12.58.16 AM.avif)

Upon investigation it appears it could one of a couple things. A cache issue, which I have cleared. Or an issue with the .htaccess file hosted on the domain provider. They make this surprisingly easy to investigate - see File Manager.

![](/public/images/posts/2024/12/Screenshot-2024-12-26-at-1.08.13 AM.avif)

cPanel! I have heard quite a bit about this but was not aware it was simply a file directory browser on the backend of the site. This file was hiding - **Settings > Show Hidden Files (dotfiles)**.

![](/public/images/posts/2024/12/Screenshot-2024-12-26-at-1.09.36 AM.avif)

> **Instructions for the .htacess file:** [https://blog.hubspot.com/website/json-response-error-wordpress](https://blog.hubspot.com/website/json-response-error-wordpress)
> 
> 1.  Connect to your server using [an FTP client](https://blog.hubspot.com/website/ftp-client?_ga=2.60068772.887187886.1640086542-680633211.1640086542&hubs_content=blog.hubspot.com/website/json-response-error-wordpress&hubs_content-cta=an%20FTP%20client) or [cPanel](https://blog.hubspot.com/website/what-is-cpanel-hosting?hubs_content=blog.hubspot.com/website/json-response-error-wordpress&hubs_content-cta=cPanel) File Manager.
> 2.  Find your .htaccess file (it should be in the root folder).
> 3.  Download a copy of the file to your computer as a backup (just in case).
> 4.  Delete the file from your server.
> 5.  Go to **Settings → Permalinks** in your WordPress dashboard and click **Save Changes**. This will force WordPress to generate a new, clean .htaccess file.
> 6.  Check if the problem is fixed.

Well, I deleted the .htaccess file and sadly the error did not resolve.

After searching for a list of possible solutions I decided to investigate conflicts with the code in my post. Reflecting on some training with SQL injection based attacks - I am assuming there is a string of text creating the issue.

## Found!

The offending string in my Wordpress post:

```
"curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar"
```

If you remove the quotation marks and put this curl request in a post, it triggers the _"Updating failed. The response is not a valid JSON response."_

Not a Java expert by any means but there is clearly an issue with parsing the curl command. It seems quotation marks must be used or the standardized # symbol. I will be sure to add a # to them in the future to stick to the comment styling.

```
# curl -O https://...
```
