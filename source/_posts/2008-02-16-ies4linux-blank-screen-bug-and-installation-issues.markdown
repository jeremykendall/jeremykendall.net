---
author: admin
comments: true
date: 2008-02-16 15:38:37+00:00
layout: post
slug: ies4linux-blank-screen-bug-and-installation-issues
title: IEs4Linux Blank Screen Bug and Installation Issues
wordpress_id: 17
categories:
- Development
- Linux
tags:
- how to
- ie
- Linux
---

If you want / need to install Internet Explorer in Linux, I highly recommend [IEs4Linux](http://www.tatanka.com.br/ies4linux/page/Main_Page) by Sergio Lopes.


> IEs4Linux is the simpler way to have Microsoft Internet Explorer running on Linux (or any OS running Wine).


I recently purchased a new [Dell 1420n with Ubuntu 7.10 pre-installed](http://www.dell.com/ubuntu).  One of the first things I wanted to do was install all of my development tools, including IEs4Linux.  What should have been an easy installation turned into a nightmare.  I kept running into python errors that aborted the install.  Once I finally got the application to install it was unusable.  The IE toolbar was missing, including the address bar, the application would frequently crash, and I experienced maddening display bugs.

Apparently the blank screen bug has been a widespread issue.  Sergio has [addressed it](http://www.tatanka.com.br/ies4linux/news/54) on his blog and pushed an emergency release of IEs4Linux, version 2.99.0.1, to resolve this issue only.

That's all well and good, but I still had the python issues to deal with.  What finally worked for me was to install IEs4Linux with the --no-gui flag, like so:

    
    $ ./ies4linux --no-gui



For more information on installing IEs4Linux, including IE versions 5 and 5.5, visit the [IEs4Linux installation page](http://www.tatanka.com.br/ies4linux/page/Installation).
