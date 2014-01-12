---
layout: post
title: "Installing phpass from Openwall via Composer"
date: 2013-12-13 13:05:29 -0600
comments: true
categories: 
    - phpass
    - Composer
    - security
    - passwords
---

**[UPDATE: Added a PHP version clarification at the end of the post.]**

Managing dependencies via [Composer](http://getcomposer.org) is one of the most
revolutionary advancements in the history of PHP. Composer packages are frequently
hosted on [Github](http://github.com), listed on
[Packagist](https://packagist.org/), and required in your project via the 
[require field](http://getcomposer.org/doc/04-schema.md#package-links) in `composer.json`.

## So Where is phpass?

What happens when that's not the case?  One library of note,
[phpass](http://www.openwall.com/phpass/), is not available on Github (or any
other supported VCS)[^1] and therefore can't simply be added to the `require`
field for easy installation. All is not lost, however, thanks to Composer's
[package repository](http://getcomposer.org/doc/05-repositories.md#package-2)
feature[^2].

## Behold, Composer's 'Package' Repository!

After reviewing the package repository docs, I found it ridiculously easy to require
phpass in my project. Here's what you have to do.

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
phpass as a project dependency.  Sweet!

**UPDATE - CLARIFICATION**: Using phpass is only advisable for PHP versions
that won't support the new password hashing functions.  That's any version of
PHP less than 5.3.7: 

* **PHP >= 5.5**: [Password hashing functions](http://us3.php.net/manual/en/ref.password.php) available natively
* **PHP >= 5.3.7, < 5.5**: [password_compat](https://github.com/ircmaxell/password_compat) provides forward compatibility
* **PHP < 5.3.7**: phpass is the gold standard

If you're at PHP >= 5.3.7, enjoy this article as a Composer tip you might not
have know about until now and use
[password_compat](https://github.com/ircmaxell/password_compat).  If you're at
PHP < 5.3.7, this is both a Composer tip and an admonition to upgrade you
password security. Do it!

*Many thanks to [Meroje](http://jeremykendall.net/2013/12/13/installing-phpass-from-openwall-via-composer/#comment-1191140568) 
and [@craig_bass](https://twitter.com/craig_bass/status/420666580129697792) for
pointing out that password_compat is superior, making it clear that I needed to
post a clarification.*

[^1]: Yes, there are phpass repos on Github, but Anthony Ferrara [recommends against them](https://twitter.com/ircmaxell/status/411566597208170496). When Anthony talks security, I listen.
[^2]: Be aware, there are significant drawbacks to this method (noted at the bottom of the [Package documentation](http://getcomposer.org/doc/05-repositories.md#package-2)), but sometimes it's the only way.
[1]: https://github.com/ircmaxell/password_compat
