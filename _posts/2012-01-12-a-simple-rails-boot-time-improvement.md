---
layout : post
title : A Simple Rails Boot Time Improvement
category : Refactoring
tags : [gem, ruby, performance, rails]
---
This simple improvement can have varying effects on the application depending on
how many Gems you require. Before you actually jump into this I would definitely
recommend having a full unit-test suite. Anytime you’re refactoring code or
changing how dependencies are loaded you want to be sure you aren’t messing
things up.

The examples from this post were performed on a Rails 3.1.3 application running
on Ruby 1.9.3 on a Mac OSX PowerBook. The Gemfile contained about 65 gems in
development mode.

The first step to any performance improvement endeavor is to benchmark. It’s
also important that you perform this step more than once – this gives you a more
consistent reading as the first run can often be inflated above the average
run-time.

{% highlight bash %}
(master) % time bundle exec rake environment
bundle exec rake environment  29.09s user 2.44s system 61% cpu 51.570 total

(master) % time bundle exec rake environment
bundle exec rake environment  28.02s user 1.77s system 98% cpu 30.184 total

(master) % time bundle exec rake environment
bundle exec rake environment  28.24s user 1.84s system 98% cpu 30.632 total
{% endhighlight %}

Now let’s take a look at your application.rb and measure how much time we’re
spending loading gems.

{% highlight ruby %}
# application.rb
require 'benchmark'

Benchmark.bm do |x|
  x.report do
    if defined?(Bundler)
      Bundler.require(*Rails.groups(:assets => %w(development test)))
    end
  end
end
{% endhighlight %}

When we run this benchmark we get an interesting result:

{% highlight ruby %}
(master) % time bundle exec rake environment
       user     system      total        real
  10.910000   0.900000  11.810000 ( 11.969414)
bundle exec rake environment  27.99s user 1.76s system 98% cpu 30.199 total

(master) % time bundle exec rake environment
       user     system      total        real
  10.770000   0.900000  11.670000 ( 11.721163)
bundle exec rake environment  27.71s user 1.78s system 99% cpu 29.721 total
{% endhighlight %}

We’re spending about 1/3 of our time in this `Bundler.require` statement. The
reason for this is that Bundler is loading and executing every gem in your
Gemfile, and with 65 gems that’s a hefty portion of time spent executing code.
We can confirm this by benchmarking the two parts of Bundler.require –
Bundler::Runtime#setup and Bundler::Runtime#require. I’m going to leave out the
details, but require is the offender here, and that makes sense. Bundler isn’t
really doing anything fancy, but it is responsible for executing all those gems
you’ve collected in your Gemfile.

Here’s a simple way to improve that.

<ol>
  <li>
Begin with a minor gem. By that I mean a gem that is only used in one or two
files or that has minimal impact on your application. That’s usually a good
starting point because the effects are minor. Also, don’t pick a gem that rails
automatically includes anyways – that won’t work.
  </li>
  <li>
Check if the gem has Railties. One way to do this is to navigate to its
source code directory and search for uses of ‘Railtie’.
{% highlight bash %}
cd `bundle list flickr_fu`
find . | xargs grep Railtie
{% endhighlight %}
  </li>
  <li>
If you see an output then that gem uses Railties and you need to move on to
another gem for this optimization.
  </li>
  <li>
If you don’t see an output then the gem does not use Railties. That means it
doesn’t hook into the Rails initialization process, which in turn means that it
does not need to be required during the application boot. So, let’s take care of
that – tell bundler to not require that gem.
{% highlight ruby %}
# Gemfile
gem 'flickr_fu', require: false
{% endhighlight %}
  </li>
  <li>
Find uses of that gem, and manually require the gem in those files.
{% highlight ruby %}
# lib/flickr_decorator.rb
require 'flickr_fu'

module FlickrDecorator
  ...
end
{% endhighlight %}
  </li>
  <li>Run your tests.</li>
  <li>Run your benchmarks.</li>
</ol>

You probably won’t notice a difference form a few measly gems, but once you’re
done with the entire process you’ll be looking at a significant time
improvement. In this particular application I was able to shave 10s off each
application boot. There’s an added benefit that you now know exactly which files
are using which gems. This makes it easier to test these files in isolation and
measure the impact/use of a gem. Enjoy!
