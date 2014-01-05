---
layout: post
title: "Installing PHPass from Openwall via Composer"
date: 2013-12-13 13:05:29 -0600
comments: true
categories: 
    - PHPass
    - Composer
---

Managing dependencies via [Composer](http://getcomposer.org) is one of the most
revolutionary advancements in the history of PHP. Composer packages are frequently
hosted on [Github](http://github.com), listed on
[Packagist](https://packagist.org/), and required in your project via the 
[require field](http://getcomposer.org/doc/04-schema.md#package-links) in `composer.json`.

## So Where is PHPass?

What happens when that's not the case?  One library of note,
[PHPass](http://www.openwall.com/phpass/), is not available on Github (or any
other supported VCS)[^1] and therefore can't simply be added to the `require`
field for easy installation. All is not lost, however, thanks to Composer's
[package repository](http://getcomposer.org/doc/05-repositories.md#package-2)
feature[^2].

## Behold, Composer's 'Package' Repository!

After reviewing the package repository docs, I found it ridiculously easy to require
PHPass in my project. Here's what you have to do.

``` javascript
{
    "repositories": [
    {
        "type": "package",
        "package": {
            "name": "openwall/phpass",
            "version": "0.3",
            "dist": {
                "url": "http://www.openwall.com/phpass/phpass-0.3.tar.gz",
                "type": "tar"
            },
            "autoload": {
                "classmap": ["PasswordHash.php"]
            }
        }
    }
    ],
    "require": {
        "openwall/phpass": "0.3"
    }
}
```

Now you can run `composer install` (or `composer update`, as appropriate) and Composer will install
PHPass as a project dependency.  Sweet!

[^1]: Yes, there are PHPass repos on Github, but Anthony Ferrara [recommends against them](https://twitter.com/ircmaxell/status/411566597208170496). When Anthony talks security, I listen.
[^2]: Be aware, there are significant drawbacks to this method (noted at the bottom of the [Package documentation](http://getcomposer.org/doc/05-repositories.md#package-2)), but sometimes it's the only way.
