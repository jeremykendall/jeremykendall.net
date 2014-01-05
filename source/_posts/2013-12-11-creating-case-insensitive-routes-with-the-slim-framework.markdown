---
layout: post
title: "Creating Case-insensitive Routes with the Slim Framework"
date: 2013-12-11 17:56:55 -0600
comments: true
categories: 
    - Slim
---

This [Stack Overflow question](http://stackoverflow.com/questions/19953870/how-to-set-up-case-insensitive-routes-with-slim-framework/20532205#20532205) 
about case-insensitive routing with the Slim Framework caught my eye recently. The question asks, in part:

> How can I avoid setting up two separate routes [with different cases] that trigger the same callback function?

I found the question intriguing because 1) I love Slim and 2) I've never really
thought about whether or not URLs are case-sensitive to begin with.  My
immediate thought was, "They're already case-insensitive! Has dude even tested this?"

## Are URLs Case-sensitive?

Well, kinda.  It frequently, but not always, boils down to whether or not the
web server's filesystem is case sensitive. The HTTP server (Apache, nginx, etc)
can get involved, as they can (always?) be configured to serve URLs with or
without regard to case sensitivity.  The web application and/or framework is
involved too.  If all other factors are case-insensitive but your application's
router is case-sensitive, then your application's URLs will be too. This is the
case with the Slim Framework.

## Should URLs Be Case-sensitive?

Again, kinda.  In "HTML and URLs", the [w3c has this to say](http://www.w3.org/TR/WD-html40-970708/htmlweb.html):

> URLs in general are case-sensitive (with the exception of machine names).
> There may be URLs, or parts of URLs, where case doesn't matter, but
> identifying these may not be easy. Users should always consider that URLs are
> case-sensitive.

*In general* and *should* are the operative words there.  It's OK either way[^1], but assuming case-sensitivity is prudent.

## So What About Slim?

URLs in the Slim Framework are case-sensitive[^2].  That's correct based on
what we learned about URL case-sensitivity from the w3c, but what if you have a
use case that requires case-insensitive URLs? What do do then?

## The Magic of Slim Hooks

What are Slim Hooks?  From the [documentation](http://docs.slimframework.com/#Hooks-Overview):

> A “hook” is a moment in the Slim application lifecycle at which a priority
> list of callables assigned to the hook will be invoked. A hook is identified
> by a string name.

Slim provides [six default hooks](http://docs.slimframework.com/#Default-Hooks), 
three invoked before the current route is dispatched and three invoked after.
By using one of the hooks that's invoked before the route is matched, we can
alter the incoming URL's path to match the case of the routes we've defined
in our Slim application.

## The Case-insensitive Route Hook

First, if there are any routes with mixed case, *change them all to lower case*.
That's the de facto standard for creating routes anyhow, and if any requests
come in that match the old, mixed case routes, we'll take care of those with
the hook in the below example.  Same issue, we're just flipping it on its head.
Seriously, don't get weird about changing those routes.

Next, register this callback on the `slim.before.router` hook:

``` php
$app->hook('slim.before.router', function () use ($app) {
    $app->environment['PATH_INFO'] = strtolower($app->environment['PATH_INFO']);
});
```

This works because Slim matches the routes you've defined against the Slim
Environment's `PATH_INFO` (originally found in `$_SERVER['PATH_INFO']`).  Since
your routes are lower case and the incoming request paths are lower case,
you've accomplished case insensitive routing in 3 lines of code.  BOOM.

[^1]: It's interesting to note that [domain names *are* case-insensitive](http://tools.ietf.org/html/rfc4343), regardless of whether or not that site's URLs are case-insensitive.
[^2]: A pull request against Slim 2.4 is in the works that will allow case-insensitive routes via a config setting.
