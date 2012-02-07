---
layout: post
title: Spork Testing Tip - Caching Classes
categories: [Tutorial]
tags: [drb, rails, spork, tdd, rspec, cucumber]
---

[Spork](https://github.com/timcharper/spork) is a great gem that runs a
distributed Ruby server which you can run your tests against. The benefit of
using a DRb server is that you don't have to wait for Rails to boot up every
time you want to run your RSpec and Cucumber tests, which means doing test or
behavior driven development with these testing frameworks in Rails is actually
possible because you can avoid the 30+ second boot times that often accompany
large Rails applications. Generally to make Spork useful you need to not cache
your classes in your testing environment by adding the following line to
environments/test.rb:

{% highlight ruby %}config.cache_classes = false{% endhighlight %}

The reason for this is that you want spork to re-load the related classes one
every test-run, so that changes that you make to your classes are re-loaded
rather than using a cached version from when Spork first booted. The downside of
this approach is that you lose the cached class performance boost to your tests
when you are running outside of the DRb server, for example in your continuous
integration environment. To remedy this simply change the configuration line to:

{% highlight ruby %}config.cache_classes = ENV['DRB'] == 'true' ? false : true{% endhighlight %}

Spork sets the `ENV['DRB']` variable to `true` when it is booted, so this line
lets you conditionally cache classes depending on if you are running in the
spork distributed ruby environment. Happy testing!

**UPDATE (June 1, 2011)**: I've written a more
[complete post on cached-classes, spork, sham and Rspec][post], which includes a
gist of the complete spec_helper.rb file, test.rb file and Gemfile.

[post]: http://ablogaboutcode.com/2011/05/18/spork-rspec-sham-and-caching-classes/
