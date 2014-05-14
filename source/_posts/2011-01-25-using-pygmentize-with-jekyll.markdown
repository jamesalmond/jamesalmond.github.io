---
title: Pygmentize errors with Jekyll
layout: post
---

I've recently switched my blog to Jekyll and I've set up the system on a
number of machines. The first machine was a seamless install but on my
second machine I can across the following error when trying to generate
pages with code using Pygments:

```
no such file or directory - pygmentize -l text -f html -O encoding=utf-8
```

It turns out I'd installed Pygments after installing the Jekyll gem
without realising. Uninstalling the Jekyll gem, installing Pygments and
reinstalling the Jekyll gem. Sorted!
