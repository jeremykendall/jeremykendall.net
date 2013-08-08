---
author: "Jeremy Kendall"
comments: true
date: 2010-04-30 16:45:31+00:00
layout: post
slug: upgrading-to-ubuntu-10-04-breaks-serving-php-from-home-directories
title: Upgrading to Ubuntu 10.04 Breaks Serving php from Home Directories
wordpress_id: 288
categories:
- Development
- Linux
tags:
- apache
- apache2
- php
- ubuntu
---

I upgraded to <a href="http://ubuntu.com">Ubuntu 10.04</a> this morning and immediately noticed I could no longer serve php applications from my home directory.  When I tried to visit one of my projects, http://test.local for example, Firefox opened a download dialog with a message similar to "You have chosen to open index.phtml . . ."

After quite a bit of googling and forum searching, I went to Twitter asking for help.  Many thanks to <a href="http://twitter.com/sypherNL">@sypherNL</a> for helping me resolve this one.

It seems that something has changed in `/etc/apache2/mods-enabled/php5.conf`.  If you're experiencing this same issue, check your `php5.conf` and see if it matches mine.

```apache
<IfModule mod_php5.c>
    <FilesMatch "\.ph(p3?|tml)$">
        SetHandler application/x-httpd-php
    </FilesMatch>
    <FilesMatch "\.phps$">
        SetHandler application/x-httpd-php-source
    </FilesMatch>
    # To re-enable php in user directories comment the following lines
    # (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it
    # prevents .htaccess files from disabling it.
    <IfModule mod_userdir.c>
        <Directory /home/*/public_html>
            php_admin_value engine Off
        </Directory>
    </IfModule>
</IfModule>
```

If your php5.conf file looks like the one above, simply comment out the ```<IfModule mod_userdir.c>``` lines as instructed.  Once that's done, restart apache with the following command:

```bash
sudo /etc/init.d/apache2 reload
```

Clear your browser's cache and try to visit your site again.  I didn't think the fix took at first, but after clearing cache everything worked just fine.

<strong>Related Links</strong>
<ul>
    <li>Thanks again to <a href="http://twitter.com/sypherNL">@sypherNL</a> for showing me how to resolve this one.</li>
    <li>Also see Marco Rodrigues' post <a href="http://marco.tondela.org/2010/03/your-public_html-with-php5-isnt-working-in-ubuntu-lucid/">Your public_html with PHP5 isn't working in Ubuntu 10.04 Lucid?</a></li>
</ul>
