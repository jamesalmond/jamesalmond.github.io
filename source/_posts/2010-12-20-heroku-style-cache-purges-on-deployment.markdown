---
title: Heroku-style cache purges on deployment with Varnish
layout: post
---

I've been using Varnish to put caches in front of service components recently and I was updating our deployment scripts to reflect that. One thing I wanted to achieve is the Heroku-style cache-flush on re-deployment. To do this we just need to add a simple line to the scripts, but before we can do that we need to enable the Varnish management interface by adding an option to the varnish startup command:

{% highlight text %}
varnishd -f /my/config.vcl -s malloc,25M -a 0.0.0.0:9090 -T 0.0.0.0:9092
{% endhighlight %}

The -T option tells varnish on which address and port to expose the management interface; in this case port 9092 on localhost. We can then send commands to the management interface. Available commands are listed as part of the varnishd documentation. The command we're interested in is "url.purge" which takes a regex parameter that matches paths to delete. If you want to purge all the cached pages then just run the following in your deploy script (probably somewhere towards the end, don't want to flush the cache on a broken deploy!):

```
varnishadm -T 0.0.0.0:9092 "url.purge .*"
```

If your cache is large then it's probably not a good idea to delete absolutely everything. For example, if you only wanted to expire all your stylesheets then you could use something like this:

```
varnishadm -T 0.0.0.0:9092 "url.purge .css$"
```

That purges every file in the cache that ends in .css. Happy caching!

