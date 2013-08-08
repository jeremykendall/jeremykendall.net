---
author: "Jeremy Kendall"
comments: true
date: 2008-04-10 11:59:11+00:00
layout: post
slug: dell-1420n-ubuntu-wireless-issue-resolved
title: Dell 1420n Ubuntu Wireless Issue Resolved
wordpress_id: 21
categories:
- Linux
tags:
- bug
- dell
- wireless
---

A few months ago I picked up a Dell 1420n loaded with Ubuntu 7.10.  I've really enjoyed the laptop, but I've experienced ongoing issues with my wireless connectivity.  The laptop will connect to my wireless router without any problems.  Staying connected to the wireless router was the problem.

After using the laptop for anywhere from 5 minutes to 36 hours, the wireless connection would drop.  The only way to reconnect was to reboot the laptop.  That got old _fast_, but being a new Linux user I had no idea how to troubleshoot the issue.  I finally Googled the problem and, via [this thread](http://ubuntuforums.org/showthread.php?t=720683) on the excellent Ubuntu Forums, I found the DellLinuxWiki and the [answer to my problem](http://linux.dell.com/wiki/index.php/Ubuntu_7.10/Issues/ipw3945_Wireless_Network_Module_Issues).

The issue has something to do with the ipw3945 wireless module.  The [DellLinuxWiki entry](http://linux.dell.com/wiki/index.php/Ubuntu_7.10/Issues/ipw3945_Wireless_Network_Module_Issues) suggests using the network module iwl3945 instead, and provides simple step-by-step instructions as to how to disable the ipw3945 module and how to enable the iwl3945 module.

The resolution works like a charm, and I haven't had a wireless connectivity issue since.
