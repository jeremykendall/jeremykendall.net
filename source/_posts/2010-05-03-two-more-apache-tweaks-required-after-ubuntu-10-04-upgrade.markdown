---
author: "Jeremy Kendall"
comments: true
date: 2010-05-03 13:30:41+00:00
layout: post
slug: two-more-apache-tweaks-required-after-ubuntu-10-04-upgrade
title: Two More Apache Tweaks Required After Ubuntu 10.04 Upgrade
wordpress_id: 296
categories:
- Development
- Linux
tags:
- apache
- apache2
- ubuntu
---

While I was troubleshooting why [Apache stopped serving php apps from my home directory](http://www.jeremykendall.net/2010/04/30/upgrading-to-ubuntu-10-04-breaks-serving-php-from-home-directories/), I ran into two more annoyances that required attention.  I figured I'd share them as well in case you run into them yourself.

Here's what I saw when I reloaded Apache:


    
    
    jkendall@san-diego:/etc/apache2$ sudo /etc/init.d/apache2 reload
      * Reloading web server config apache2
    apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
    [Fri Apr 30 13:47:38 2010] [warn] NameVirtualHost *:80 has no VirtualHosts
                                                                                        [ OK ]
    



As you can see, Apache reloaded fine, but I don't like seeing anything other than `[ OK ]` when I'm reloading my web server.  Let's tackle these one at a time.

**Could not reliably determine the server's fully qualified domain name**

Why is Apache all of a sudden complaining about what's worked for so long?  I'm not sure exactly, but thankfully the fix was simple and easy.  Simply adding "`ServerName localhost`" to `/etc/apache2/httpd.conf` took care of the first complaint (Big thanks to Mohamed Aslam for [his clear instructions](http://mohamedaslam.com/how-to-fix-apache-could-not-reliably-determine-the-servers-fully-qualified-domain-name-using-127011-for-servername-error-on-ubuntu/) on how to get this fixed.).

**NameVirtualHost *:80 has no VirtualHosts**

The problem boiled down to having `NameVirtualHost` defined in more than one place.  In my case, `NameVirtualHost` was defined both in `/ect/apache2/ports.conf` and `/etc/apache2/sites-available/default`.  Commenting out the `NameVirtualHost *:80` line in `/etc/apache2/sites-available/default` did the trick (Thanks to the guys in [this Server Fault thread](http://serverfault.com/questions/1405/apache2-startup-warning-namevirtualhost-80-has-no-virtualhosts), especially to [Ivan](http://serverfault.com/users/942/ivan), for providing the necessary clues to track this one down.).

After making the above changes, reloading Apache didn't throw any more warnings. w00t!
