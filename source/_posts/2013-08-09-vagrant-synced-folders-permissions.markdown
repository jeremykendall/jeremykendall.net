---
layout: post
title: "Vagrant Synced Folders Permissions"
date: 2013-08-09 07:32
comments: true
categories: ["Vagrant", "Development", "VirtualBox"]
---

Having trouble getting your Synced Folders permissions just right in your
Vagrant + VirtualBox VM? They've been giving me some grief lately. Here are the 
(undocumented) `Vagrantfile` options that finally got it sorted out.

### Permissions Challenges

The issue that got me digging into this was trying to get permissions just so
to allow my web apps to write logs.  As you probably know, apache (or your web
server of choice), sometimes needs write access to certain web application
directories.  I've always take care of that by adding the apache user group to
the directories in question and then giving that user full access.  Doing that
in Ubuntu looks something like:

``` bash
chown -R jeremykendall.www-data /path/to/logs
chmod 775 /path/to/logs
```

If you've tried doing something similar in your Vagrant shared folders, you've
likely failed.  This, as it turns out, [doesn't work with VirtualBox shared folders](https://github.com/mitchellh/vagrant/issues/897)
-- you have to make the changes in your `Vagrantfile`.

### Setting Permissions via the Vagrantfile

Here's my new `synced_folder` setting in my `Vagrantfile`:

``` ruby
  config.vm.synced_folder "./", "/var/sites/dev.query-auth", id: "vagrant-root", 
    :owner => "vagrant", 
    :group => "www-data", 
    :extra => "dmode=775,fmode=664" 
```

I'm sure you can immediately see what resolved the issue.  Lines 2 and 3 set
the owner and group, respectively, and line 4 sets directory and file modes
appropriately.  That simple fix was frustratingly difficult because I couldn't
find it documented *anywhere*. After much searching and opening far too many
browser tabs, I cobbled together the info above.  A quick `vagrant reload`
later and I was off to the races.
