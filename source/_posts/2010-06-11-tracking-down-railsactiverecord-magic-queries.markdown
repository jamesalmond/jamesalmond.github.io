---
title: Tracking down Rails/ActiveRecord magic queries with QueryTrace
layout: post
---

I'm not going to open the can of worms that is "Rails Magic (TM)", but sometimes you need to track down why Rails is doing something that you don't expect it to.

A form helper I was using was unexpectedly loading all objects from the database rather than just the ones I passed it. Turns out I was actually passing it a nil object, but whilst I was investigating I came across the neat plugin QueryTrace.

QueryTrace gives you log output with a backtrace of where the query came from, allowing you to track down rogue queries:

```
Night Load (2.5ms)   SELECT * FROM `nights` WHERE (`nights`.room_type_id = 1) ORDER BY date ASC
    app/helpers/admin/nights_helper.rb:68:in `nights_form_helper'
    app/views/admin/nights/edit.html.haml:13
    haml (3.0.10) rails/./lib/haml/helpers/action_view_mods.rb:225:in `call'
    haml (3.0.10) rails/./lib/haml/helpers/action_view_mods.rb:225:in `form_for_without_live_validations'
    haml (3.0.10) lib/haml/helpers.rb:588:in `call'
    haml (3.0.10) lib/haml/helpers.rb:588:in `haml_bind_proc'
    ....
    dragonfly (0.6.1) lib/dragonfly/middleware.rb:13:in `call'
```

It is installed as a plugin using
```
script/plugin install git://github.com/ntalbott/query_trace.git
```

As this [Pivotal Labs blog post suggests](http://pivotallabs.com/users/alex/blog/articles/300-rake-query-trace), the plugin can also be useful for load or performance but may slow down the rest of your app whilst it is being run.
