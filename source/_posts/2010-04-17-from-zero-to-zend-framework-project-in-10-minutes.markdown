---
author: "Jeremy Kendall"
comments: true
date: 2010-04-17 19:18:46+00:00
layout: post
slug: from-zero-to-zend-framework-project-in-10-minutes
title: From Zero to Zend Framework Project in 10 Minutes
wordpress_id: 243
categories:
- Development
tags:
- apache
- Linux
- ubuntu
- zend framework
- zend tool
---

I first started working with the [Zend Framework](http://framework.zend.com) in July of 2007.  The framework has come a long way since then, and one of my favorite new components is [Zend Tool](http://framework.zend.com/manual/en/zend.tool.html).  Until Zend Tool came on the scene, my least favorite part of any Zend Framework project was getting the project up and running, from zero to "Hello World," if you will.  I always left out some important piece of configuration, screwed up the directory structure, or made some other equally foolish, simple mistake that kept me chasing my tail until I finally got everything set up just right.  No longer! Zend Tool does all of that work for me, allowing me to get to work on the meat of my project right away.

Of course, you can't use Zend Tool until you've got the framework installed.  Once the framework is installed, there are very specific steps required to fire up Zend Tool.  Once Zend Tool builds the project for you, you're going to need a [virtual host](http://httpd.apache.org/docs/2.0/vhosts/) set up so that you can actually run the thing. No problem, right?  Well, maybe not.

It can be really challenging for the new guy to get from zero to a Zend Framework project without wanting to chunk his computer out the window ([headdesk](http://www.urbandictionary.com/define.php?term=headdesk&defid=2569861) anyone?).  I know it was that way for me.  In the interest of saving his sanity and helping the new gal get going, I've tried to put all the necessary steps together in one place.

Here's what we're going to do:

* Install the Zend Framework
* Get Zend Tool up and running
* Create a new Zend Framework project
* Get an virtual host up and running
* Celebrate victory!

**Disclaimer**

While I'm going try to stay true to my "Zero to Zend Framework Project" thesis, I am making a few assumptions.  This tutorial is written for [Ubuntu](http://www.ubuntu.com/), [PHP 5](http://php.net/), and [Apache](http://httpd.apache.org/). While I'd love to be able to address Windows, Mac, and other \*nix distros, I can only share what I know.  I'm also assuming PHP 5 (PHP 5.2.4 at minimum) and Apache are both installed and functioning correctly, and Apache's [mod_rewrite](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html) module is up and running.  If that's not the case, please refer to the resources section at the end of this post to find instructions on getting everything ready to go.

**Installing the Zend Framework**

Installing the Zend Framework is as easy as downloading a .zip or .tar.gz file (your choice) and extracting the contents onto your machine.

* Create a directory named phplib in your home folder.
* Head over to [Zend Framework: Downloads](http://framework.zend.com/download/latest) and download Zend Framework <version_num> Full.
* Double click on the downloaded file, choose "Extract," navigate to the phplib directory, and click "Extract."

As of this writing, the most recent version of the Zend Framework is 1.10.3, so the full path to my Zend Framework installation would be

```    
    /home/jkendall/phplib/ZendFramework-1.10.3
```

**IMPORTANT**: Make sure that you downloaded Zend Framework <version_num> Full.  That will be important later.

**TIP**: As new versions of the framework are released, I like to be able to switch between them easily.  I always make a [soft link (symbolic link, symlink)](http://en.wikipedia.org/wiki/Symbolic_link) to the latest release and name it Zend.  You can either right-click on the ZendFramework-1.10.3 folder and select "Make Link" (make sure to name the link "Zend"), or create a soft link from the command line like so:

```bash    
    ~/phplib/ZendFramework-1.10.3$ ln -s ZendFramework-1.10.3 Zend
```

When a new version of the framework is released, I install the new version to its own folder and switch the soft link to target the directory containing the latest release.  That can come in handy down the road.

**Get Zend Tool running**

There are a [few different ways to get Zend Tool running](http://framework.zend.com/manual/en/zend.tool.usage.cli.html), perhaps better than mine, but I like to create a [bash alias](http://www.sucka.net/2009/09/ubuntu-command-line-tip-1-%E2%80%93-aliases/) for zf.sh (For information on how to get bash aliases working, see this [bash aliases tutorial](http://www.sucka.net/2009/09/ubuntu-command-line-tip-1-%E2%80%93-aliases/).).  My alias looks like `alias zf='/usr/share/phplib/Zend/bin/zf.sh'`.

Whichever method you use, make sure to test your alias by calling `zf --help` from the command line.

**Creating a new Zend Framework Project**

Now we're getting to the good stuff.  First, head back to your home directory and create a new directory called public_html.  This is where you're going to store your Zend Framework project (and any future web projects, for that matter).

Next, cd into the public_html directory and execute the following command:

```bash  
    ~/public_html$ zf create project ZeroToZF
```

That's all there is to it!  You've got your project structure in place, a default IndexController and ErrorController, your application config, the necessary view scripts, etc (See the [Zend Application Quick Start](http://framework.zend.com/manual/en/zend.application.quick-start.html) for full rundown of what got created.).

**IMPORTANT**: While the full project structure is now in place, the Zend Framework **is not included in your project for you**.  Deciding how the framework should be included in the project is up to the developer.  I like to copy the library out of the Zend Framework install folder into the application's library folder.  In our example, I would copy

```    
    /home/jkendall/phplib/ZendFramework-1.10.3/library/Zend
```

into

```  
    /home/jkendall/public_html/ZeroToZF/library
```

When done properly, the full path to Zend/Log.php in your application should be

```  
    /home/<username>/public_html/ZeroToZF/library/Zend/Log.php
```

Sweet!  We're almost there.

**Creating a Virtual Host**

There are a decent number of steps here, but executing them is straightforward, so bear with me.  Also, a lot of these steps need to be executed from the command line.  [Forewarned is forearmed](http://idioms.thefreedictionary.com/Forewarned+is+forearmed) and all that.

If you have not done so already, open the terminal application (Applications -> Accessories -> Terminal) and cd into `/etc/apache2/sites-available`.  Here you'll find virtual host definitions.  Most likely you'll see a file named default (or possibly 000-default).  Peek at that if you'd like to see an example of a virtual host, but what we're going to put together is a lot simpler.

**IMPORTANT**: Since `/etc` and its subfolders are owned by root, you'll need to run all of the following commands as root.  We'll be using both the [sudo](https://help.ubuntu.com/community/RootSudo#sudo) and the [gksudo](https://help.ubuntu.com/community/RootSudo#Graphical sudo) commands to do that.

After using the command line to cd into `/etc/apache2/sites-available`, execute the following command to create your own vhost file:

```  
    /etc/apache2/sites-available$ gksudo gedit ZeroToZf.local
```

The file extension isn't important, a lot of people use .conf, but I like to use .local for sites hosted locally.

Once you've got gedit open, the file's contents should look like this:

```  
    <VirtualHost *:80>
            ServerName zerotozf.local
            DocumentRoot /home/jkendall/public_html/ZeroToZF/public
    </VirtualHost>
```

Of course, jkendall should be replaced by your username.

Next we need to make your new virtual host file available to Apache.  You can do that by executing

```  
    /etc/apache2/sites-available$ sudo a2ensite ZeroToZF.local
```

If everything worked properly, you should see the following message:

```    
    Enabling site ZeroToZF.local.
    Run '/etc/init.d/apache2 reload' to activate new configuration!
```

Next, edit your `/etc/hosts` file by adding `zerotozf.local` like so:

```  
    /etc/apache2/sites-available$ gksudo gedit /etc/hosts
```

Add the following line below the entry for localhost

```  
    127.0.0.1 zerotozf.local
```

**NOTE**: When adding a host to /etc/hosts, the case of the hostname is not important.  I like to add hostnames in lowercase, but you can add it as ZeroToZF.local if you like.

Save the file, close gedit, and execute

```  
    /etc/apache2/sites-available$ sudo /etc/init.d/apache2 reload
```

You should see the message

```bash  
    * Reloading web server config apache2                                          [ OK ]
```

Open up your browser and visit [http://zerotozf.local](http://zerotozf.local).  You should see the Zend Framework welcome screen.

Congrats!  You did it!

**Next Steps**

Now that you've got your first project up and running, you're probably going to want to actually do something with it.  I'd highly recommend heading over to [Rob Allen's site](http://akrabat.com/) for his excellent [Getting Started with Zend Framework](http://akrabat.com/zend-framework-tutorial/) tutorial.  It's the same tutorial I followed when I first started with the framework (an earlier version, of course), and I've referred back to it many times since.

**Wrapping Up**

Getting started with the Zend Framework was a challenging proposition for me.  Just getting to the point where I could start working on a project was sometimes maddening as a result of all the steps involved, many of which had nothing to do with the framework, at least not directly.  I've tried, hopefully successfully, to lay out all the steps you might need to get from zero to a Zend Framework project in (about) 10 minutes.

Working with the Zend Framework has been a rich and rewarding experience for me.  I've learned _almost_ everything I know about best practices, object oriented programming, \*nix, and Apache (to name just a few) as a direct or indirect result of the Zend Framework.  I wouldn't have been able to do that without the help of the Zend Framework community.  I won't name names, as I'm sure to unintentionally leave out some great folks, so I'll throw you a link to the [Zend Framework Community Forum](http://n4.nabble.com/Zend-Framework-Community-f634137.html).  Head over there when you've got a Zend Framework problem you just can't solve on your own.  You'll meet some great folks and learn a lot in the process.  They've saved my bacon more than once.

If I've left out any steps or made some egregious error, please let me know in the comments.  I'll be grateful and post updates and corrections as soon as possible.

**Resources**

* How to [install PHP in Ubuntu](https://help.ubuntu.com/9.10/serverguide/C/php5.html)
* How to [install Apache in Ubuntu](https://help.ubuntu.com/9.10/serverguide/C/httpd.html)
* [Enabling mod_rewrite](http://www.ghacks.net/2009/12/05/enable-mod_rewrite-in-a-ubuntu-server/)
* A great [bash aliases](http://www.sucka.net/2009/09/ubuntu-command-line-tip-1-%E2%80%93-aliases/) tutorial for Ubuntu
* Setting up [Zend Tool on Windows](http://www.armando.ws/2009/05/how-to-set-up-zend_tool-on-windows/comment-page-1/)
* The [Zend Framework Community Forum](http://n4.nabble.com/Zend-Framework-Community-f634137.html) on Nabble
* [Getting Started with Zend Framework](http://akrabat.com/zend-framework-tutorial/) by Rob Allen
