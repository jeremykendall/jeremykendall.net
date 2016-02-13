---
layout: post
title: "Vagrant Synced Folders Permissions"
date: 2013-08-09 07:32
comments: true
categories: ["Vagrant", "Development", "VirtualBox"]
---

*UPDATE: Since writing this post Vagrant has changed the synced folder settings
and I've come across a new (and better?) way of handling this problem. Scroll
down for the updates.*

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

*UPDATE: Thanks to Joe Ferguson for [pointing out in the comments](http://jeremykendall.net/2013/08/09/vagrant-synced-folders-permissions/#comment-1118614212)
that Vagrant has been upgraded and my example was no longer current.  Below are
both examples marked by Vagrant version.*

Here's my new `synced_folder` setting in my `Vagrantfile`:

Vagrant v1.1+:

``` ruby
  # Vagrant v1.1+
  config.vm.synced_folder "./", "/var/sites/dev.query-auth", id: "vagrant-root",
    owner: "vagrant",
    group: "www-data",
    mount_options: ["dmode=775,fmode=664"]
```

Vagrant 1.0.x:

``` ruby
  # Vagrant v1.0.x
  config.vm.synced_folder "./", "/var/sites/dev.query-auth", id: "vagrant-root", 
    :owner => "vagrant", 
    :group => "www-data", 
    :extra => "dmode=775,fmode=664" 
```

I'm sure you can immediately see what resolved the issue.  Lines 3 and 4 set
the owner and group, respectively, and line 5 sets directory and file modes
appropriately.  That simple fix was frustratingly difficult because I couldn't
find it documented *anywhere*. After much searching and opening far too many
browser tabs, I cobbled together the info above.  A quick `vagrant reload`
later and I was off to the races.

### UPDATE: Alternate Method

An alternate method that doesn't include modifying your synced folder permissions is
changing the web user to the vagrant user.  Bad idea?  Security problem?  Not on your dev
VM it ain't, and that's good enough for me. Big thanks to Chris Tankersley for all
the help getting this one figured out.

{% twitter oembed https://twitter.com/JeremyKendall/status/383416513715523584 %}

Chris and I both put together gists, and [this is how I'm currently doing it](https://github.com/jeremykendall/flaming-archer/blob/develop/manifests/default.pp#L41-L53)
in Flaming Archer, but probably the best method for changing the apache user
to the vagrant user comes from the [Intracto Puppet apache manifest](https://github.com/Intracto/Puppet/blob/master/apache2/manifests/init.pp).

```
# Source https://raw.github.com/Intracto/Puppet/master/apache2/manifests/init.pp

# Change user
exec { "ApacheUserChange" :
    command => "sed -i 's/APACHE_RUN_USER=www-data/APACHE_RUN_USER=vagrant/' /etc/apache2/envvars",
    onlyif  => "grep -c 'APACHE_RUN_USER=www-data' /etc/apache2/envvars",
    require => Package["apache2"],
    notify  => Service["apache2"],
}

# Change group
exec { "ApacheGroupChange" :
    command => "sed -i 's/APACHE_RUN_GROUP=www-data/APACHE_RUN_GROUP=vagrant/' /etc/apache2/envvars",
    onlyif  => "grep -c 'APACHE_RUN_GROUP=www-data' /etc/apache2/envvars",
    require => Package["apache2"],
    notify  => Service["apache2"],
}
```

Additionally, if you're copying and pasting from anywhere, don't forget to change
the apache lockfile permissions:

```
# Source https://github.com/Intracto/Puppet/blob/master/apache2/manifests/init.pp

exec { "apache_lockfile_permissions" :
    command => "chown -R vagrant:www-data /var/lock/apache2",
    require => Package["apache2"],
    notify  => Service["apache2"],
}
```

### ACL on Shared Folders That Are Not NFS

One of the reasons the above methods are necessary is that you can't use ACLs
on shared directories.  If none of the above options appeal to you, it's possible
to use ACLs on your VM as long as the directories aren't shared. For more information,
see [Frank Stelzer's comment](https://github.com/puphpet/puphpet/issues/138#issuecomment-25434178)
regarding `setfacl` on a Vagrant box.
