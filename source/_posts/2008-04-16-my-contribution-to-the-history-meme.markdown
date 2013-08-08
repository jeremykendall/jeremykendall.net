---
author: "Jeremy Kendall"
comments: true
date: 2008-04-16 16:41:32+00:00
layout: post
slug: my-contribution-to-the-history-meme
title: My Contribution to the History Meme
wordpress_id: 23
categories:
- Linux
- Personal
tags:
- meme
---

From my work machine:
    
    jkendall@ventura:~$ uname -a
    Linux ventura 2.6.22-14-generic #1 SMP Tue Feb 12 07:42:25 UTC 2008 i686 GNU/Linux
    jkendall@ventura:~$ history | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head
    83 exit
    70 clear
    58 cd
    47 ls
    44 ps
    35 svn
    32 tail
    32 jgrep
    23 kill
    11 sshdev
    


jgrep is a bash alias for `'grep --color -r -n --exclude=\*.svn\*'`, while sshdev is a bash alias that gets me to the dev box at work without having to type too much.

See more examples [here](http://www.thelins.se/johan/2008/04/history-meme.html), [here](http://www.0xdeadbeef.com/weblog/?p=356), [here](http://www.j5live.com/2008/04/15/history-meme/), and [here](http://blogs.gnome.org/thos/2008/04/10/history-meme/).
