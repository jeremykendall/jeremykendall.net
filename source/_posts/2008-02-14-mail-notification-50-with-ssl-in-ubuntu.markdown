---
author: "Jeremy Kendall"
comments: true
date: 2008-02-14 22:24:22+00:00
layout: post
slug: mail-notification-50-with-ssl-in-ubuntu
title: Mail Notification 5.0 with SSL in Ubuntu
wordpress_id: 8
categories:
- Linux
tags:
- how to
- ssl
- ubuntu
---

**The Problem**

We recently began using SSL to connect to IMAP at work.  Prior to switching to SSL I had been using the excellent [Mail Notification](http://www.nongnu.org/mailnotify/) to let me know if I had any messages in my inbox.   As soon as we switched over to SSL, Mail Notification quit alerting me to the presence of new email.

I figured that all I had to do was change my preferences in Mail Notification and select SSL.  Turns out I was right, except I couldn't select SSL - all of the "SSL/TSL" options were grayed out.  Why in the world would that be?  After some research I discovered that:

* SSL isn't available if the package was built without SSL support (makes sense)
* The OpenSSL license conflicts with the Debian license

Mail Notification _is_ available in the Ubuntu repository, but without SSL support.  Bummer.

**Build it Yourself**

The resolution is to build Mail Notification with SSL support enabled, but I quickly discovered that building Mail Notification was not as easy as I thought it would be.  Although the installation is well documented in the INSTALL file, I still ran into a lot of problems with dependencies.

**Dependencies Required**

After a lot of troubleshooting and not a little frustration, I came up with a list of dependencies that I needed installed in order to configure and make Mail Notification 5.0 on Ubuntu Gutsy:
  
* build-essential
* gnome-core-devel
* libnotify-dev
* libgnome2-dev
* libgmime-2.0-2-dev
* libssl-dev

I used the Synaptic Package Manager to install each of these dependencies.  Read on if you're looking to troubleshoot a specific error that you're running into.

**Errors and Troubleshooting**

The first time I ran  
    
```bash
$ ./configure
```

I got this error:

```bash
checking for C compiler default output file name... 
configure: error: C compiler cannot create executables
See `config.log' for more details.
```

Installing **build-essential** took care of the C compiler issue, but I found a new one the next time I ran configure:

```bash
Error: checking for GNOME... no
configure: error: unable to find the GNOME libraries
```

This one drove me a little batty.  What does it mean it can't find the GNOME libraries?  I'm running GNOME for Pete's sake!  After a decent amount of hair pulling and a seemingly endless amount of Googling, I finally found that **gnome-core-devel**, **libnotify-dev**, and **libgnome2-dev** resolved the `unable to find the GNOME libraries` error.

Next up was the GMIME error:

```bash
checking for GMIME... configure: error: Package requirements (gmime-2.0 >= 2.1.0) were not met:
No package 'gmime-2.0' found
```

At least by now I was making it most of the way through the `configure` process.  I found that installing **libgmime-2.0-2-dev** resolved the issue, finally allowing me to complete the configuration.

Of course, the whole point was to build Mail Notify with SSL support.  What do you think I found when configure finally ran all of the way through?  Down at the bottom of the options list, I saw this:

```bash
--enable-ssl                 no (OpenSSL not found)
```

By now, I had started seeing a pattern: look up the dependencies, find their dev libraries, install their dev libraries, and voila, on to the next issue.  With that in mind, I installed **libssl-dev** and ran 
    
```bash
$ ./configure
```

one last time.  Mail Notify configured without errors and with SSL support.  A quick `make` and `make install` later and I had a Mail Notify 5.0 installation complete with SSL support.

**Other Options**

There seem to be a lot of different ways to skin this particular cat.  The solution above is what worked for me, your mileage may vary.  You may find it useful to refer to the discussion in this [related bug report](https://bugs.launchpad.net/ubuntu/+source/mail-notification/+bug/44335) for background.  

For another way to resolve this issue, you might try "[How to make a small change to a Debian tool and repackage it.](http://www.howtoforge.com/repackage_deb_packages_debian_ubuntu)"  I didn't find this article until after I had resolved the issue myself, but it looks like it might be a lot simpler.
