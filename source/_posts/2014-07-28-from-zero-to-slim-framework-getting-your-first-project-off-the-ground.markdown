---
layout: post
title: "From Zero to Slim Framework: Getting Your First Project Off The Ground"
date: 2014-07-28 11:19:01 -0500
comments: true
categories: ["PHP", "Slim"]
---

When I was a brand new web developer, I was overwhelmed by the amount of general 
knowledge required to get a project off the ground: Web server (as in
configuring a server OS), web server (as in nginx or Apache), PHP installation,
PHP configuration, application configuration, and so forth.  I was willing and
able to learn, but even the best blog posts and documentation frequently assumed a certain level of
existing knowledge, much of which I didn't have. My goal with this post is to help you
get your first [Slim Framework](http://www.slimframework.com/) project started
*without* assuming any knowledge on your part. We'll literally go from
absolutely nothing to a functioning [Slim-Skeleton](https://github.com/codeguy/Slim-Skeleton) 
application.

### Requirements

While I'm not going to make any general knowledge assumptions (call me out in
the comments if I miss the mark), I am going to set a few requirements.  In
short, we'll be using virtualization technology ([Vagrant](http://www.vagrantup.com/) 
and [VirtualBox](https://www.virtualbox.org/)), the [Ubuntu](http://www.ubuntu.com/) 
14.04 LTS operating system, and [Composer](https://getcomposer.org/), a dependency 
management tool for PHP.

### Why Do This All "By Hand"?

While there's an "easier, softer way" (think [Phansible](http://phansible.com/) or 
[PuPHPet](https://puphpet.com/)), I think it's important to know what's going
on behind the scenes.  What if you need to fix something on your server once
it's in production?  What if you have to tweak a setting here or there, or
create a new vhost or nginx site?  It's a good idea to have done it at least
once by hand before moving on to automated solutions.  You'll have a better
feel for what's happening, why it's happening, and how to fix any problems that
might arise in the future.

### Preparing the Host Environment

By "host", I mean your computer.  These are the first steps we'll take to prepare your
computer to host the virtual machine we'll use to run the tutorial code.

 * Install Vagrant: Grab an installer for your OS [here](http://www.vagrantup.com/downloads.html).
 * Install VirtualBox: Grab an installer for your OS [here](https://www.virtualbox.org/wiki/Downloads).

### Create Your VM

 * Create a directory for your project, and then change into your project directory
 * Make a sub-directory for your Slim project within your new project directory: `mkdir zero-to-slim.dev`
 * Now run `vagrant init ubuntu/trusty64` from your project directory

`vagrant init` will create a `Vagrantfile`, the file that tells Vagrant how to
build your VM.  We'll need to edit that file and add a few settings.

Edit `/path/to/project/Vagrantfile` and add the following lines after `config.vm.box`:

``` ruby
  config.vm.hostname = "zero-to-slim.dev"
  config.vm.network :private_network, ip: "192.168.56.103"
  config.vm.synced_folder "./zero-to-slim.dev", "/var/www/zero-to-slim.dev", id: "web-root",
      owner: "vagrant",
      group: "www-data",
      mount_options: ["dmode=775,fmode=664"]
```

Save and close your `Vagrantfile`, and then run `vagrant up`.  You'll be treated
to some output as your VM is built.  Once that's done, the VM is ready to configure.

### Connect and Configure VM

Run `vagrant ssh` from your project directory.  The following steps will be
completed on the VM, not your host machine.

> If you're on Windows, the `ssh` command won't be available to you.  You'll need 
> to use a program like [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/) 
> instead. Here are some [step-by-step instructions](https://github.com/Varying-Vagrant-Vagrants/VVV/wiki/Connect-to-Your-Vagrant-Virtual-Machine-with-PuTTY) 
> for configuring your Windows box to connect to your new VM.

 * `sudo apt-get update`
 * `sudo apt-get install curl vim wget python-software-properties -y`
 * `sudo add-apt-repository ppa:ondrej/php5 -y`
 * `sudo apt-get update`

### Choose a Web Server and a PHP Version

> You must choose one or the other, not both. nginx is extremely
> popular, but if you're more comfortable with Apache I've included instructions
> below.

#### nginx and PHP-FPM

 * `sudo apt-get install php5-fpm php5-cli php5-xdebug nginx -y`

#### Apache2 and PHP

 * `sudo apt-get install php5 php5-cli php5-xdebug apache2 -y`

### Install Composer Globally

See Composer's [Getting Started](https://getcomposer.org/doc/00-intro.md)
section for the most up-to-date installation instructions (There's an 
[installer available](https://getcomposer.org/doc/00-intro.md#installation-windows) 
for those of you on Windows). Below is how I install Composer on both Mac and Linux.

 * `curl -sS https://getcomposer.org/installer | php`
 * `sudo mv composer.phar /usr/local/bin/composer`

### Use Composer to Install Slim Skeleton

Heads up, this is going to take a while.  Composer is an amazing technology,
but in this case it's pretty slow (this has more to do with the VM than with
Composer). Run the command, grab a cup of coffee, and come on back to finish
up.

 * `composer create-project slim/slim-skeleton /var/www/zero-to-slim.dev`

> If you suspect there's a problem with the `composer create-project` command
> due to how long it takes to get feedback, you can add the verbose option
> (`-vvv`) to the command, like so: `composer create-project slim/slim-skeleton /var/www/zero-to-slim.dev -vvv`

### Configure Your Web Server

Since the following files aren't in a synced folder, which would allow you to edit them 
from your host machine, and since you need to be root to edit them, the best
way to do so is directly on the server using a text editor. I've chosen
[Vim](http://www.vim.org/) in this case. Once again, here's an opportunity to
learn about a tool that you may find yourself needing at some point in the
future, even if you only ever use it to edit a file or two on your server(s).

#### Editing a Config File with Vim

 * Type `sudo vim <server_config_file>` (either `/etc/nginx/sites-available/default` or
   `/etc/apache2/sites-available/000-default.conf`, depending)
 * Type `gg` on the keyboard to ensure you're at the top of the file
 * Delete everything in the file with `dG`
 * Type `i` to enter insert mode
 * Copy the appropriate config from below and paste it into the config document
 * Hit `ESC` to exit insert mode
 * Type `:wq` to write your changes and quit the file.

Congrats! You've just edited your server config using Vim, an accomplishment in itself.

#### nginx and PHP-FPM

Replace the contents of `/etc/nginx/sites-available/default` with the following:

``` nginx
server {
    listen      80;
    server_name zero-to-slim.dev;
    root        /var/www/zero-to-slim.dev/public;

    try_files $uri /index.php;

    # this will only pass index.php to the fastcgi process which is generally safer but
    # assumes the whole site is run via Slim.
    location /index.php {
        fastcgi_connect_timeout 3s;     # default of 60s is just too long
        fastcgi_read_timeout 10s;       # default of 60s is just too long
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        include fastcgi_params;
    }
}
```

Save and close the default config, and then restart nginx: `sudo service nginx restart`

#### Apache2

Replace the contents of `/etc/apache2/sites-available/000-default.conf` with the following:

``` apache
<VirtualHost *:80>
    DocumentRoot "/var/www/zero-to-slim.dev/public"
    ServerName zero-to-slim.dev

    <Directory "/var/www/zero-to-slim.dev/public">
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

* Save and close the default config
* Enable `mod_rewrite` for URL rewriting: `sudo a2enmod rewrite`
* Restart Apache: `sudo service apache2 restart`

### Configure PHP for Dev Environment

Since editing `php.ini` with Vim would require quite a few detailed
instructions, I've opted to show you how to override `php.ini` settings by
adding an additional config file.

On your host machine, create a file in your project directory name `00-php.ini`.
Paste the following contents into the file:

```
error_reporting = -1
display_errors = On
display_startup_errors = On
html_errors = On
```

These settings ensure that PHP will report any and all errors it encounters,
and that PHP will display those errors on screen.  The `html_errors` setting
will provide a nice looking error formatted by `xdebug`.

Since your project directory is synced to `/vagrant` on your VM, the file you just
created will be available on your VM.  Copy the new config file into the directory 
scanned by PHP for additional config files and restart your web server. The
following commands should be executed on your VM, not your host machine.

#### nginx and PHP-FPM

* `sudo cp /vagrant/00-php.ini /etc/php5/fpm/conf.d/`
* `sudo service php5-fpm restart`

#### Apache2 and PHP

* `sudo cp /vagrant/00-php.ini /etc/php5/apache2/conf.d/`
* `sudo service apache2 restart`

### Final Configuration

 * Add `192.168.56.103 zero-to-slim.dev` to `/etc/hosts` on the host machine 
   (Windows users, see: [How do I modify my hosts file](http://www.rackspace.com/knowledge_center/article/how-do-i-modify-my-hosts-file))
 * Visit [http://zero-to-slim.dev](http://zero-to-slim.dev) in your web browser

You should see the Slim welcome page.  If you don't, there should be an error 
displayed telling you exactly what went wrong.  Double check the steps above to make 
sure you didn't miss anything. If you still have problems, drop me a line in the comments and 
I'll help you get them sorted out.

### Wrapping Up

So there you have it.  You started with nothing at all and now have a working
Slim Framework application. Congrats!  

FYI, I highly recommend basing your Slim apps on the Slim Skeleton.  Install
it, modify it for your specific application needs, and you'll have a much
easier time getting up and running, I promise. That's what I did when I first
started with Slim.

> Next time we'll talk about how to automate this process.  Now that you know what's 
> involved, there's no point in wasting precious dev time manually configuring
> new machines for each new project.
