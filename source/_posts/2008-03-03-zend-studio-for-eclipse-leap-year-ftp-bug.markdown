---
author: "Jeremy Kendall"
comments: true
date: 2008-03-03 15:41:12+00:00
layout: post
slug: zend-studio-for-eclipse-leap-year-ftp-bug
title: Zend Studio for Eclipse Leap Year FTP Bug
wordpress_id: 19
categories:
- Development
tags:
- bug
- zend studio
---

I found a bug in [Zend Studio for Eclipse](http://www.zend.com/en/products/studio/) the other day.  I created a new QA subdomain for one of my web project.  When I tried to view the new directory in Zend Studio for Eclipse, it wouldn't show up.  Nothing that I did in Zend Studio would make it show up.




It drove me nuts until I found a clue in [this discussion](http://www.zend.com/forums/index.php?t=msg&th=5930&start=0&S=cd5c05e96b6390777cd3fa8a6d2af137) on the Zend Forums.  Zend Forum member adlorenz posted the clue, saying




. . . it seems like **bug concerns only files/directories that have leap day date of last modification **. . .




Turns out that any directory or file created or last modified on Feb 29 of this year won't show up in FTP Remote Systems connections in Zend Studio for Eclipse.  I logged into my account, touched the directories and files that were dated Feb 29 (`$ touch dirname`), and I could finally see the directories and files in the Remote Systems Explorer.
