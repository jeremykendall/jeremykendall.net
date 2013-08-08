---
author: "Jeremy Kendall"
comments: true
date: 2009-10-28 04:12:17+00:00
layout: post
slug: network-monitoring-and-logging-with-perl
title: DIY Network Monitoring and Logging with Perl
wordpress_id: 223
categories:
- Development
- Linux
tags:
- att
- dsl
- perl
---

**The Problem**

I've been having trouble with my AT&T; DSL installation here at the new place.  My internet connection will come and go, seemingly at random, and for random amounts of time.  I [tweeted about it](http://twitter.com/JeremyKendall/status/5193232511) once already, hoping that sharing my frustration with the world might make me feel a little better.  Nope.  Didn't work.

I've had this problem before with AT&T;, when I got DSL installed at my last place.  Things were rough for a while, and then somehow they seemed to straighten out on their own.  I'm not crossing my fingers that I'll have such luck again.

**What's Really Going On?**

I decided to try and log the bounces so that I could get a better feel for what was going on.  I couldn't find exactly what I wanted online (and I couldn't always get online), so I whipped up a [Perl](http://en.wikipedia.org/wiki/Perl) script to keep an eye on my connection for me.


    
    
    #!/usr/bin/perl
    
    #
    # Test AT&T; DSL connectivity. If network is down, log to file
    # Will use log info as ammo when I call tech support.
    #
    
    use warnings;
    use strict;
    use Log::Handler;
    use Net::Ping;
    use Sys::HostIP;
    
    my $log = Log::Handler->new();
    
    $log->add(
        file => {
            filename => "/var/log/testNetwork.log",
            mode     => "append",
            maxlevel => "info",
            minlevel => "warning",
        }
    );
    
    my $ipAddress          = Sys::HostIP->ip; 
    my $matchHomeNetworkIp = ($ipAddress =~ /^192\.168\.10\.\d{1,3}$/);
    
    if (!$matchHomeNetworkIp) {
        $log->debug("Current IP $ipAddress does not appear to be on your home network. Exiting");
        exit;
    }
    
    my $router = "192.168.10.1";
    my $modem  = "192.168.1.254";
    my $host   = "www.yahoo.com";
    
    my $p = Net::Ping->new("icmp");
    
    if (!$p->ping($router)) {
        $log->warning("Router $router is unreachable. Exiting.");
        exit;
    }
    
    if (!$p->ping($modem)) {
        $log->warning("Modem $modem is unreachable. Exiting.");
        exit;
    }
    
    if ($p->ping($host, 2)) {
        $log->info("$host is reachable");
    } else {
        $log->warning("$host is NOT reachable");
    }
    
    $p->close();
    



**Code Review**

While the code is simple, there are a couple of things to note.  First, you'll notice I've baked my router's IP range into a [regex](http://en.wikipedia.org/wiki/Regular_expressions) that will tell me if I can even get that far.  If not, there's no point in checking anything else, so I exit.

Next I fire up [Net::Ping](http://search.cpan.org/~smpeters/Net-Ping-2.36/lib/Net/Ping.pm).  While there are six different options for Net::Ping->new(), the only one that worked for me was icmp.  Icmp requires root privileges to run, so keep that in mind. 

Next I [ping](http://en.wikipedia.org/wiki/Ping) important parts of the home network.  I want to be sure that my wireless router and the DSL modem are both online before I try and ping an external site.

If everything looks good on the inside, I'll ping an external site and see if I get anything back.  You'll notice that I've added a second parameter to ping this time.  That's the number of seconds I'll wait for a response before failing (of course, it is possible that Yahoo! won't respond to a ping request here and there, making it appear as if the network is down, but I'm not so worried about a false positive or two).

**Implementation**

Now that I've got this nifty script to tell me about the health of my network, I need a way to run it on a regular basis.  It's important to run it as root because of the icmp ping.  [Cron](http://en.wikipedia.org/wiki/Cron) was the obvious solution, as it's purpose is to run commands on a predetermined schedule. As long as I add the script to root's cron file, it'll run with root permissions.  Sweet!

Adding my script to the root cron file was as easy as issuing 
    
    sudo crontab -e

and adding the following line:


    
    
    */1 * * * * perl /home/jkendall/dev/perl/util/testNetwork.pl
    



With my cron file in place, my script fires off every minute.  As long as I'm on my home network, I'll get a log entry telling me whether or not the network is up.

**Keeping an Eye on the Results**

Now that my script is actually doing something, I need to be able to parse the results.  A quick 
    
    tail -f /var/log/testNetwork.log

allows me to keep track of the results as they come in.  I'm also using grep to pull all of the "NOT reachable" lines out of the log file with 
    
    grep -n --color "NOT reachable" /var/log/testNetwork.log

 I could always just open the log file and read through it from top to bottom, but what fun is that?  A decent tool to parse the fail log is going to be necessary, but I haven't whipped it up yet. 

**Teh Suck**

As I've been writing this blog post, I've been running this script in the background.  Here are the results of an hour or so of logging (all times are CDT):


    
    
    Oct 27 21:03:33 [INFO] www.yahoo.com is reachable
    Oct 27 21:04:03 [WARNING] Router 192.168.10.1 is unreachable. Exiting.
    Oct 27 21:05:02 [INFO] www.yahoo.com is reachable
    Oct 27 21:06:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:07:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:08:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:09:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:10:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:11:02 [INFO] www.yahoo.com is reachable
    Oct 27 21:12:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:13:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:14:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:15:32 [INFO] www.yahoo.com is reachable
    Oct 27 21:16:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:17:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:18:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:19:11 [INFO] www.yahoo.com is reachable
    Oct 27 21:20:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:21:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:22:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:23:03 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:24:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:25:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:26:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:27:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:28:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:29:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:30:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:31:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:32:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:33:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:34:21 [INFO] www.yahoo.com is reachable
    Oct 27 21:35:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:36:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:37:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:38:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:39:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:40:12 [INFO] www.yahoo.com is reachable
    Oct 27 21:41:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:42:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:43:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:44:31 [INFO] www.yahoo.com is reachable
    Oct 27 21:45:01 [INFO] www.yahoo.com is reachable
    Oct 27 21:46:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:47:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:48:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:49:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:50:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:51:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:52:42 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:53:41 [WARNING] www.yahoo.com is NOT reachable
    Oct 27 21:54:01 [INFO] www.yahoo.com is reachable
    



Weak sauce, man.  Weak sauce.

**Wrapping Up**

Having this script handy doesn't make the [fail](http://failblog.org/) any better, but at least I know a little more about what's happening.  I'd like to think it'll give me some leverage when I try to get this worked out with AT&T;, but who knows.  I'll let you know how it goes.

[**Update**: post title change to better reflect content]
