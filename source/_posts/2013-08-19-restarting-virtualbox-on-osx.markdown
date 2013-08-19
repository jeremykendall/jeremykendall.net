---
layout: post
title: "Restarting VirtualBox on OSX"
date: 2013-08-19 17:51
comments: true
categories: ["VM", "Vagrant", "VirtualBox"]
---

My Vagrant + VirtualBox VM workflow was distrupted late this afternoon by
an error during `vagrant up`:

``` bash
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'precise64'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

    Command: ["hostonlyif", "create"]

    Stderr: 0%...
    Progress state: NS_ERROR_FAILURE
    VBoxManage: error: Failed to create the host-only adapter
    VBoxManage: error: VBoxNetAdpCtl: Error while adding new interface: failed to open /dev/vboxnetctl: No such file or directory

    VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component HostNetworkInterface, interface IHostNetworkInterface
    VBoxManage: error: Context: "int handleCreate(HandlerArg*, int, int*)" at line 68 of file VBoxManageHostonly.cpp
```

Googling for `Failed to create the host-only adapter` didn't seem to return anything
useful, so I tried with `failed to open /dev/vboxnetctl` and immediately found an answer.

The trick, according to GitHub user [lslucas](https://github.com/lslucas), is to 
[restart VirtualBox from the command line](https://github.com/mitchellh/vagrant/issues/1671#issuecomment-22146167)
like so:

``` bash
 $ sudo /Library/StartupItems/VirtualBox/VirtualBox restart
```

That worked like a champ for me, and I was immediately able to get back to work.

For what it's worth, here's my current software config:

* VirtualBox 4.2.16
* Vagrant 1.2.2
* Mac OS X 10.8.4
