---
title: Caching multiple backends on the same server with Varnish
layout: post
---

I've been looking at taking our caching practices past the simple drop-in solution of rack-cache today after we had some problems with caching asynchronous requests. As long as we don't need detailed per-app configuration and can just rely on simple cache expiry headers there are some benefits of running a ubiquitous caching layer over our apps. After having previously seen it used on Heroku, I took a look at Varnish, the "web accelerator". It's fairly easy to get set up, so I won't go into it here, just take a look at the tutorials.

One thing that I wanted to do that I couldn't see an obvious solution for was running a caching layer for multiple apps that can be accessed on different ports. Varnish is generally started using something similar to the following command:

```
varnishd -f /my/config/dir/varnish.vcl -s malloc,50M -a 0.0.0.0:80
```

Which starts a varnish instance accessible on port 80. Running this command again with a different port and different config doesn't seem to start a new Varnish process. To get around this all you have to do is name your Varnish process (and put it on a different port!):

```
varnishd -f /my/config/dir/varnish.vcl -s malloc,50M -a 0.0.0.0:80 -n app_2
```

```
varnishd -f /my/config/dir/varnish_alternative.vcl -s malloc,50M -a 0.0.0.0:9090 -n app_1
```

Now we can add as many caches as we want! Maybe I missed something in the docs, but it took me a while to work this out, hope it helps.

