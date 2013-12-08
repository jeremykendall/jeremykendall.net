---
layout: post
title: "Bypassing Guzzle Cache: CallbackCanCacheStrategy"
date: 2013-12-07 00:36
comments: true
categories: ["Guzzle", "caching"]
---

I'm a big fan of the [Guzzle PHP HTTP client](http://docs.guzzlephp.org/).
I use it whenever I need to make requests of 3rd party APIs from my applications.
If you're still writing cURL requests by hand or have rolled your own HTTP
client, I highly recommend checking out Guzzle.

I'm currently making heaviest use of Guzzle in my photo-a-day project,
[Flaming Archer](https://github.com/jeremykendall/flaming-arche://github.com/jeremykendall/flaming-archer),
in order to get photo data from Flickr. To keep from hammering the 
[Flickr API](http://www.flickr.com/services/api/), I'm caching all of those 
requests. Guzzle makes caching ridiculously easy by way of their plugin system
and their [HTTP Cache plugin](http://docs.guzzlephp.org/en/latest/plugins/cache-plugin.html).

The problem with the caching plugin, at least at first blush, is how to bypass
the cache in certain specific instances where caching might not be appropriate.
The [docs are a little light in this area](http://docs.guzzlephp.org/en/latest/plugins/cache-plugin.html#custom-caching-decision), 
so it took me a few minutes to get it sorted out. Let's start at the top.

### The Guzzle Client

> "Clients create requests, send requests, and set responses on a request
> object. When instantiating a client object, you can pass an optional "base
> URL" and optional array of configuration options."

Here's an example of creating a Guzzle Client, based on my use case of making
requests against the Flickr API.

``` php
use Guzzle\Http\Client;

$client = new Client('http://api.flickr.com');
$client->setDefaultOption('query', array(
    'api_key' => 'EXAMPLE_API_KEY',
    'format' => 'json',
    'nojsoncallback' => 1,
));
```

I use the client for the `GET` requests I need to make against the Flickr API.  Each request will include
the above default options in the query string.  Nice!

### Adding Caching

Since I don't want to hammer the crap out of the Flickr API and start hitting
the rate limit[^1], I wanted to cache each request. Thankfully, Guzzle has an
awesome plugin system that includes an 
[HTTP Cache plugin](http://docs.guzzlephp.org/en/latest/plugins/cache-plugin.html).

> "Guzzle can leverage HTTP's caching specifications using the
> Guzzle\Plugin\Cache\CachePlugin. The CachePlugin provides a private
> transparent proxy cache that caches HTTP responses."

Rather than rolling my own caching strategy (My first solution was to write a
[decorator](http://en.wikipedia.org/wiki/Decorator_pattern) for caching), 
I decided to use Guzzle's native plugin and leave all the caching work to them.

``` php
use Guzzle\Cache\Zf2CacheAdapter;
use Guzzle\Plugin\Cache\CachePlugin;
use Guzzle\Plugin\Cache\DefaultCacheStorage;
use Zend\Cache\Backend\TestBackend;

$backend = new TestBackend();
$adapter = new Zf2CacheAdapter($backend);
$storage = new DefaultCacheStorage($adapter);
$cachePlugin = new CachePlugin($storage);

$client->addSubscriber($cachePlugin);
```

The cache plugin will now intercept and cache `GET` and `HEAD` requests made by the
client.

### CallbackCanCacheStrategy

So what if, now that you're caching each `GET` request, there's a request or
requests you don't want cached? Guzzle makes solving that problem trivial by
allowing for "[custom caching decisions](http://docs.guzzlephp.org/en/latest/plugins/cache-plugin.html#custom-caching-decision)",
but the documentation on how to make those custom decisions is 
[decidedly light](http://docs.guzzlephp.org/en/latest/plugins/cache-plugin.html#custom-caching-decision).

> ". . . you can set a custom can_cache object on the constructor of the
> CachePlugin and provide a Guzzle\Plugin\Cache\CanCacheInterface object. You
> can use the Guzzle\Plugin\Cache\CallbackCanCacheStrategy to easily make a
> caching decision based on an HTTP request and response."

Wat? 

That was clear as mud to me, so I spent a few minutes digging through the
source. This is what I came up with: 

* The `CallbackCanCacheStrategy` provides a
method of providing callbacks to the cache plugin that, based on a boolean
response, determine whether or not a particular request or response should be
cached.  
* The `CallbackCanCacheStrategy` accepts two optional arguments to its
constructor: a callable that will be invoked for requests and a callable that
will be invoked for responses.  The request callback gets an instance of
`Guzzle\Http\Message\RequestInterface`, and the response callback gets an
instance of `Guzzle\Http\Message\Response`.

### Bypassing Cache

In my case, I want to cache everything except for calls to the 
[flickr.photos.search](http://www.flickr.com/services/api/flickr.photos.search.html) 
API method. Since all of the `GET` requests I'm making include a `method` query string param, 
it was trivial to write the callback that got me where I needed to go.

``` php
$canCache = new CallbackCanCacheStrategy(
    function ($request) { 
        if ($request->getQuery()->get('method') === 'flickr.photos.search') {
            return false; 
        }

        return true;
    }
);
```

### Putting It All Together

Now that I've built my `$client`, `$storage`, and `$canCache` strategy, here's
how I put it all together.

``` php
$cachePlugin = new CachePlugin(array(
    'can_cache' => $canCache,
    'storage' => $storage,
));

$client->addSubscriber($cachePlugin);
```

Now all of my `GET` requests are cached *except* for those using the `flickr.photos.search` method. BOOM.

[^1]: I can't find the documentation on API rate limiting right now, but I know it's limited and I don't want to hit that limit.
