---
layout: post
title: "Composer Platform Packages"
date: 2014-03-08 18:36:22 -0600
comments: true
categories: ["composer", "php"]
---

Here's something about Composer that I can never remember, I always have to
look up, and I always have a hard time finding where it is in the
documentation. Ladies and gentlemen, I give you [platform packages][1]:

> ## Platform packages
> 
> Composer has platform packages, which are virtual packages for things that are
> installed on the system but are not actually installable by Composer. This
> includes PHP itself, PHP extensions and some system libraries.
> 
> * `php` represents the PHP version of the user, allowing you to apply
>   constraints, e.g. `>=5.4.0`. To require a 64bit version of php, you can
>   require the `php-64bit` package.
> 
> * `hhvm` represents the version of the HHVM runtime (aka HipHop Virtual
>   Machine) and allows you to apply a constraint, e.g., '>=2.3.3'.
> 
> * `ext-<name>` allows you to require PHP extensions (includes core
>   extensions). Versioning can be quite inconsistent here, so it's often
>   a good idea to just set the constraint to `*`.  An example of an extension
>   package name is `ext-gd`.
> 
> * `lib-<name>` allows constraints to be made on versions of libraries used by
>   PHP. The following are available: `curl`, `iconv`, `icu`, `libxml`,
>   `openssl`, `pcre`, `uuid`, `xsl`.
> 
> You can use `composer show --platform` to get a list of your locally available
> platform packages.

So that's the relevant portion of the documentation, and its `composer show --platform` that lists local platform packages. Maybe *now* I'll remember.

[1]: https://getcomposer.org/doc/02-libraries.md#platform-packages
