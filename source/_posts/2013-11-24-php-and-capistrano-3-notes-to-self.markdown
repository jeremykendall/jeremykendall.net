---
layout: post
title: "PHP and Capistrano 3: Notes To Self"
date: 2013-11-24 07:43
comments: true
categories: ["PHP", "Capistrano", "Capistrano3"]
---

I spent quite a bit of my day yesterday trying to work out a painless,
scripted, idiot-proof deployment process with [Capistrano](http://www.capistranorb.com/) for my 
[photo-a-day website](http://365.jeremykendall.net/).  I've been doing a lot of work on the
site lately, which means a lot of deployments, and I've been very unhappy with
myself for implementing what amounts to "deployment worst practices" when it
comes to my personal projects.

The last time I worked with Capistrano was about two years ago, and a lot has changed
since then.  Capistrano v3 [was released in June of 2013](http://www.capistranorb.com/2013/06/01/release-announcement.html)
and brought with it a lot of great changes, but for a guy who doesn't know ruby and 
relies on tutorials and Stack Overflow questions for help, the version bump brought
a lot of pain as well.

## Challenges

### Every Tutorial is Wrong

Just know going into this that (almost) [every tutorial you find](https://www.google.com/search?q=php+capistrano+v3&oq=php+capistrano+v3&aqs=chrome..69i57j69i64.3471j0j7&sourceid=chrome&espv=210&es_sm=91&ie=UTF-8) 
is going to be a Capistrano v2 tutorial.  Enough has changed between v2 and v3
to make those tutorials just misleading enough to cause a good amount of pain.  

### Stack Overflow is a Capistrano v3 Desert

As of this writing, there are [ten questions tagged capistrano3](http://stackoverflow.com/questions/tagged/capistrano3) 
on Stack Overflow. Seriously. Ten. And only four of those include accepted answers.  

### Capistrano v3 Documentation is Lacking

The documentation available for v3 is seriously lacking, although the problem
is more one of quantity than quality. [What's available is good](http://www.capistranorb.com/), 
there's just nowhere near enough of it.

### Caveat Emptor

These are indeed "Notes to Self".  I hope they help you out, but if they don't,
I'm giving you fair warning.  Please feel free to add what's missing to the
comments.

*Reading the *[Capistrano](http://www.capistranorb.com/) *docs is highly
recommended. These notes are supplemental.*

## PHP + Capistrano v3

*NOTE: You can find the application source, which now includes Capistrano v3,
[on GitHub](https://github.com/jeremykendall/flaming-archer).*

### Ruby

First, I had to reinstall [rvm](https://rvm.io/rvm/install), a ruby version manager.
What I don't know about installing a ruby dev environment could fill books, so I let
rvm take care of that for me.

```
\curl -L https://get.rvm.io | bash -s stable --ruby
```

(Capistrano **requires Ruby 1.9 or newer**, so the current stable Ruby from rvm
will work fine.)

### Installation

[Install Capistrano](http://www.capistranorb.com/documentation/getting-started/installation/).
There are a few options, but I used `gem install`.

```
gem install capistrano
```

**NOTE**: All of the PHP tutorials I found instruct you to install the
`railsless-deploy` gem. As Capistrano v3 "doesn't ship with any Railsisms", 
this is no longer necessary and the [railsless-deploy project is obsolete](https://github.com/leehambley/railsless-deploy).

*I'll probably go back and add a* `Gemfile` *since I'll be adding the composer gem and want everything I need in one place.*

### Preparing Your App

The `capify` command is no longer, it's now `cap install`.  

```
cd /path/to/project
cap install
```

The documentation on this portion, ["Preparing Your Application"](http://www.capistranorb.com/documentation/getting-started/preparing-your-application/),
is one of the places the documentation shines, IMHO.

### Capistrano Files

* `Capfile`: Kind of like a bootstrap. Takes care of necessary required
configs and globs for custom tasks.
* `config/deploy.rb`: Common items across all deployments go here
* `config/deploy/{staging, production}.rb`: Environment specific deployment settings

### Roles

I'm still not 100% clear on this, but roles don't seem to be roles in the ACL
sense, but rather roles in the "division of server responsibility" sense, hence the 
roles `:web`, `:app`, and `:db`.

[The docs say](http://www.capistranorb.com/documentation/getting-started/preparing-your-application/#toc_3) that you can dump the `:app` and `:db` roles if you like, but if
you're going to use the `:linked_files` and `:linked_dirs` feature (which are
pretty cool) you'll need to leave the `:app` role in place. I'm obviously doing
something wrong or missing something here, but that's what I had to do.

I found it extremely helpful to refer to the [`deploy.rb.erb` template](https://github.com/capistrano/capistrano/blob/master/lib/capistrano/templates/deploy.rb.erb).
I removed most of the example text before realizing I needed to use some of it,
and referencing the template was nice.

### SSH Forwarding

SSH Agent Forwarding is the way to go here, IMHO.  I already had a key agent running
so the [forwarding was dead simple](http://www.capistranorb.com/documentation/getting-started/authentication-and-authorisation/#toc_3),
almost like magic.  If you don't already have an agent running,
[here's some good info](https://help.github.com/articles/working-with-ssh-key-passphrases) 
from GitHub Help.

### Server Config

The section on how to set up the proper Capistrano directories on your server is
[way down deep in the Authorisation portion of the docs](http://www.capistranorb.com/documentation/getting-started/authentication-and-authorisation/#toc_8).
Short version: you need two dirs in the root of your project (on the server):
`releases` and `shared`, and they must be readable and writeable by both the
webserver and deploy user.

*Getting permissions right is easy for some, but I always return to the* 
[Setting Up Permissions](http://symfony.com/doc/current/book/installation.html#configuration-and-setup)
*section of the Symfony2 docs to get them set up properly.*

### Making Composer Work

Once I got my `deploy.rb` and `deploy/production.rb` right and tested (by deploying, natch),
I needed to create a task to run [`composer install`](http://getcomposer.org).
Getting that right turned out to be pretty difficult because of a design decision
in SSHKit.

Short story: [No spaces in command line commands](https://github.com/capistrano/capistrano/issues/719#issuecomment-26917090).

I finally got my composer command running [by doing this](https://github.com/jeremykendall/flaming-archer/blob/dc1900aab5386b164430b001032b310f78a3c0e5/config/deploy.rb#L18-L27):

```
desc 'composer install'
task :composer_install do
    on roles(:web) do
        within release_path do
            execute 'composer', 'install', '--no-dev', '--optimize-autoloader'
        end
    end
end

after :updated, 'deploy:composer_install'
```

* The `within release` bit in the command tells the task to `cd` into the 
`release` directory before running composer.
* There are `before` and `after` hooks you can apply to the deploy flow. The final line above is a hook that runs after `deploy:updated`.

### Composer Gem for Capistrano v3

Of course, as soon as I got it working, Peter Mitchell sent me this tweet:

{% twitter oembed https://twitter.com/peterjmit/status/404324240339771392 %}

I haven't yet replaced my hacked version with the gem, but I'll use it
as soon as I do any refactoring.

### Deployment Annoyances

`cap production deploy --dry-run` never worked for me, although `cap production deploy`
worked fine.  No idea why, who cares at this point.  Maybe I'll dig in later.

Also, `cap production deploy` *really* wants to run `deploy:restart`, even
though it doesn't show up anywhere in the [deploy flow](http://www.capistranorb.com/documentation/getting-started/flow/). 
I replaced the `:restart` task that's in the default `deploy.rb` template, made
sure it was empty, and deploy finally worked.

### Linked Files and Directories

The `:linked_files` and `:linked_directories` feature is really nice. I've used
it for logs, a `local.php` config, Twig caching, my SQLite database, and my
generated rss file. The linked items are for files and dirs that should be
shared between deploys.

Also, those files and dirs need to be present in the shared dir before
deploying. Deploy will puke if they're not present.

### Capistrano Variables

I couldn't find a listing of these anywhere, but `release_path` is one of them,
and it points to the latest release path.

## That's All For Now

I hope the notes help when you get ready to write your own Capistrano v3 scripts
for use with PHP.  I'll update this as I learn more, and I'll make sure to point out
any excellent point made in the comments.
