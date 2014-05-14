---
title: Spork doesn't pick up some file changes
layout: post
---

I've been using spork to speed up my tests for a while now, and I like it. But today I noticed that changes in my files weren't being picked up by RSpec or Cucumber.

The spork Github page suggests running "spork --diagnose" to find out what's being loaded on startup by spork and therefore what's not and will be monitored.

The culprit was a factory_girl factory that specified the class using the class constant, rather than a string, which meant the class was loaded before forking and any changes in the file weren't picked up thereafter.

``` ruby
# before
Factory.define :base_post, :class => Post do |h|
  h.name "Demo"
  ...
end

#after
Factory.define :base_post, :class => "Post" do |h|
  h.name "Demo"
  ...
end

```

Changing the class to a string meant the Post class wasn't picked up in the pre-fork code and future changes were picked up.
