---
author: jeremykendall
comments: true
date: 2013-07-12 14:21:06+00:00
layout: post
slug: the-composer-kerfuffle
title: The Composer Kerfuffle
wordpress_id: 467
categories:
- Development
- PHP
---

Wednesday night I discovered, much to my shock and dismay, that [Composer](http://getcomposer.org/)'s install command now defaults to installing development dependencies along with a project's required dependencies.  That discovery prompted me to start a "[small twitter shitstorm](http://seld.be/notes/composer-installing-require-dev-by-default)" with [this tweet](https://twitter.com/jeremykendall/status/355148025393446913):

{% twitter oembed https://twitter.com/jeremykendall/status/355148025393446913 %}

Before I delve into why I'm shocked and dismayed, I want to say a few things about Composer and the Composer team.  In all of my years of programming in PHP, I'm not sure there's been a more important, more game changing, or more exciting project than Composer.  Being able to easily manage project dependencies has revolutionized the way I develop.  Composer, and the related [packagist.org](https://packagist.org/), have been a larger quality-of-life improvement for me than any other tool I've added to my toolkit over the years.  I'd like to extend my sincerest thanks to [Nils Adermann](http://naderman.de/), [Jordi Boggiano](http://nelm.io/jordi) and the many [community contributors](https://github.com/composer/composer/graphs/contributors) who have worked so hard and so diligently to make Composer a reality.

**My Beef with the Change**

**1. There was never any public discussion about the change**

Beyond a few brief asides in a couple of github [issues](https://github.com/composer/composer/issues/1005) and [pull requests](https://github.com/composer/composer/pull/1833), I can't find anywhere this change discussed publicly.  For a project of this size and this importance, removing the community from the decision making process was a terrible mistake.  I'd much prefer to have argued all of this before the change than after.

Yesterday, Jordi posted ["Composer: Installing require-dev by default"](http://seld.be/notes/composer-installing-require-dev-by-default) to explain his rationale for the change.  One of his points is that [he made a note in the 1.0.0-alpha changelog](https://github.com/composer/composer/releases/tag/1.0.0-alpha7) regarding the upcoming change.  While this is true, I find it insufficient.  I rarely read changelogs (my bad), but I certainly don't read changelogs to discover what's coming in the future.  That's what road maps, blog posts, and PRs are for.  Putting that note in the changelog was "too little, too early".

**2. Composer philosophy and workflow has always been, "install for production, update for development"**

The longstanding Composer rule of thumb has always been "install for production, update for development".  [Adam Brett's post on Composer workflow](http://adamcod.es/2013/03/07/composer-install-vs-composer-update.html) and the difference between update and install is a good example of this rule of thumb.  Humorously enough, Jordi reinforces that rule of thumb (while defending the change that composer update installs dev requirements by default) in his [blog post](http://seld.be/notes/composer-an-update-on-require-dev) _immediately prior_ to the post defending the composer install change:



> "The install command on the hand remains the same. It does not install dev dependencies by default, and it will actually remove them if they were previously installed and you run it without --dev. Again this makes sense since in production you should only run install to get the last verified state (stored in composer.lock) of your dependencies installed."



That rule of thumb is now turned on its head, and the default "composer install in production" advice now needs updating, careful warnings, and caveats.

**3. Tools should never default to dev (unless they're meant for dev, of course)**

This is a philosophical point on my part, but it's one I don't think is unique to me and it's one that I think can be well defended.  My point here is that one should always write code and tools in such a way that deployments to production will only ever result in production code being deployed.  Clear as mud?  Let me try with an example.

I frequently use environment variables to allow my applications to detect which environment they're running in.  If those environment variables don't exist, then the application should default to production.  Why?  Because dev environments are the special case, not production, and it's far too easy to forget to add those environment variables when deploying.  I make my life easier by making the production environment as idiot-proof as possible, and not the other way around.

This philosophy was in place in the prior behavior of composer install (and composer update, for that matter).  Now that it's changed, the production environment is far more likely to suffer than the development environment.  Forgetting to add the --dev flag in development is a lot less (potentially) costly than forgetting to add the --no-dev flag in production.

In a seeming contradiction, I've said that [I have no problem with composer update defaulting to installing dev dependencies](https://twitter.com/jeremykendall/status/355186613376126976).  I've gone back and forth on that a bit when considering my "tools should never default to dev" position, but I don't think I'm being inconsistent here.  Since the rule of thumb encourages using update in development and _never_ in production, then update becomes a dev tool which can safely default to installing development dependencies.  Having said that, if consistency between commands is important, then composer update should no longer default to dev and the changes to both install and update should be reverted.

**In Closing**

Composer has become an integral part of my workflow, and a critical piece of the PHP development process in general.  I loved Composer before this change and I'll love Composer after.  That said, changing the Composer command that is intended primarily for production use is extremely disruptive and a very bad call, especially considering how the change came about.
