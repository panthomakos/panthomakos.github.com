---
layout: post
title: Ruby 1.9.3-p0 Released
categories: [News]
tags: [rails, ruby, rvm]
---

Among the smattering of new methods and bug fixes the most important improvement
in Ruby 1.9.3 release, for Rails users, is going to be in application boot time.
I've seen at least 50% performance boosts in load time simply by upgrading from
Ruby 1.9.2. The installation was painless although I had some issues compiling
the [ffi](https://rubygems.org/gems/ffi) gem extensions. I can't wait to migrate
production code over to this new version and development is going to be so much
more of a pleasure.

To upgrade with RVM:

{% highlight bash %}
rvm get head
rvm reload
rvm install 1.9.3
{% endhighlight %}

You can read the
[news post](http://svn.ruby-lang.org/repos/ruby/tags/v1_9_3_0/NEWS) or
[changelog](http://svn.ruby-lang.org/repos/ruby/tags/v1_9_3_0/ChangeLog) for
additional details.

In particular I found the following of interest.

  * `Random.rand` now accepts a range as an argument `Random.rand(10..20)`.
  * `IO.write` and `IO.binwrite` for writing to files in normal and binary mode.
  * New Encodings including `UTF-16` and `UTF-32`.
  * ARGF new methods like `ARGF.write` and `ARGF.puts`.

Enjoy!
