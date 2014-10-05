---
layout: post
title: "Ubuntu 14.04 Gearman Config Bug"
date: 2014-09-04 07:48:10 -0500
comments: true
categories: ["devops", "gearman", "ubuntu"]
---

While configuring [Gearman][1] (for async jobs here at Graph Story), I ran
across a bug that causes Gearman to ignore its config file. The bug shows up on
Ubuntu 14.04 Trusty Tahr in Gearman version 1.0.6-3 installed from the Ubuntu
repositories.

Full disclosure: I can only confirm the existence of this bug in the above
mentioned environment.  I have not tested for the bug in any other
configuration.

## Bug Report - TL;DR

There is a [bug report][2] filed by Artyom Nosov which details the issue and
includes a patch.  The [patch diff][3] is linked from the [bug report][2] and 
is included below:

``` diff
diff -urN gearmand-1.0.6-old/debian/gearman-job-server.upstart gearmand-1.0.6/debian/gearman-job-server.upstart
--- gearmand-1.0.6-old/debian/gearman-job-server.upstart2013-11-11 01:58:42.000000000 +0400
+++ gearmand-1.0.6/debian/gearman-job-server.upstart20132013-12-13 22:30:32.392281779 +0400
@@ -9,4 +9,7 @@
 
  respawn
   
   -exec start-stop-daemon --start --chuid gearman --exec /usr/sbin/gearmand -- --log-file=/var/log/gearman-job-server/gearman.log
   +script
   +    . /etc/default/gearman-job-server
   +    exec start-stop-daemon --start --chuid gearman --exec /usr/sbin/gearmand -- $PARAMS --log-file=/var/log/gearman-job-server/gearman.log
   +end script
```

## The Fix

Gearman runtime configuration is handled by `/etc/default/gearman-job-server`
in Ubuntu.  The default upstart job does not use the Gearman config file,
causing Gearman to run with its defaults, which may or may not be acceptable to
you. To resolve the issue, update `/etc/init/gearman-job-server.conf` using the
diff included above.  For reference, here is my entire
`gearman-job-server.conf` file:

```
# -*- upstart -*-

# Upstart configuration script for "gearman-job-server".

description "gearman job control server"

start on (filesystem and net-device-up IFACE=lo)
stop on runlevel [!2345]

respawn

# exec start-stop-daemon --start --chuid gearman --exec /usr/sbin/gearmand -- --log-file=/var/log/gearman-job-server/gearman.log

# PATCH: https://bugs.launchpad.net/ubuntu/+source/gearmand/+bug/1260830
script
    . /etc/default/gearman-job-server
    exec start-stop-daemon --start --chuid gearman --exec /usr/sbin/gearmand -- $PARAMS --log-file=/var/log/gearman-job-server/gearman.log
end script
```

## The Benefit

Gearman will now respect the settings found in
`/etc/default/gearman-job-server`, allowing you to configure Gearman with
[command line options][4] that you'd otherwise have to add to the init script.
In the case of the Graph Story implementation, I'm binding Gearman to localhost
and using MySQL as a persistent queue. Here's what that config file looks
like:

```
# This is a configuration file for /etc/init.d/gearman-job-server; it allows
# you to perform common modifications to the behavior of the gearman-job-server
# daemon startup without editing the init script (and thus getting prompted by
# dpkg on upgrades).  We all love dpkg prompts.

# Examples ( from http://gearman.org/index.php?id=manual:job_server )
#
# Use drizzle as persistent queue store
# PARAMS="-q libdrizzle --libdrizzle-db=some_db --libdrizzle-table=gearman_queue"
#
# Use mysql as persistent queue store
# PARAMS="-q libdrizzle --libdrizzle-host=10.0.0.1 --libdrizzle-user=gearman \
#                       --libdrizzle-password=secret --libdrizzle-db=some_db \
#                       --libdrizzle-table=gearman_queue --libdrizzle-mysql"
#
# Missing examples for memcache persitent queue store...

# Parameters to pass to gearmand.
PARAMS="--listen=localhost \
        -q mysql \
        --mysql-host=localhost \
        --mysql-port=3306 \
        --mysql-user=redacted \
        --mysql-password=redacted \
        --mysql-db=gearman \
        --mysql-table=gearman_queue"
```

## Side Note and Pro Tip: MySQL Persistent Queue

Creating the MySQL database and providing privileges to your database user allows 
Gearman to create its own `gearman_queue` database table.  Don't risk creating an incompatible 
database table; let Gearman do that work for you.

[1]: http://gearman.org/
[2]: https://bugs.launchpad.net/ubuntu/+source/gearmand/+bug/1260830
[3]: https://launchpadlibrarian.net/159679541/gearman-job-server.upstart.patch
[4]: http://gearman.org/manual/job_server/#options
