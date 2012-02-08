---
layout: post
title: Spork, RSpec, Sham and Caching Classes
categories: [Tutorial]
tags: [tdd, drb, rspec, ruby, rails, sham, spork]
---

This is my favorite testing trifecta:

  * [RSpec](https://github.com/rspec/rspec) as the testing framework.
  * [Spork](https://rubygems.org/gems/spork) as the Distributed Ruby environment.
  * [Sham](https://github.com/panthomakos/sham) to create model factories.

The problem I ran into recently though was that when running RSpec tests using
Spork, changes to my models were not causing the files to re-load in the DRB
environment.  This was a big problem - it meant that the benefits provided by
Spork were virtually non-existent because every file change forced me to re-boot
my Spork server. To solve this problem I released a new version of Sham (0.5.0)
and made some changes to my spec_helper.rb and test.rb files. Here is what I
ended up with:

<ol>
<li>
Ensure your Gemfile is up to date, and update your bundle.

{% highlight ruby %}
group :test do
  gem 'rspec-rails'
  gem 'spork'
  gem 'sham'
end
{% endhighlight %}
</li>
<li>
Update test.rb to ensure that you aren't caching classes when using Spork.

{% highlight ruby %}
config.cache_classes = !(ENV['DRB'] == 'true')
{% endhighlight %}
</li>
<li>
Modify your spec_helper.rb file, adding the Spork.each_run block.

{% highlight ruby %}
require 'spork'

Spork.prefork do
  ...
end

Spork.each_run do
  ActiveSupport::Dependencies.clear
  ActiveRecord::Base.instantiate_observers
  Sham::Config.activate!
end if Spork.using_spork?
{% endhighlight %}
</li>
<li>
You're done! <a href="https://gist.github.com/979222">Here's a complete gist</a>
of the files.
</li>
</ol>

The Spork.each_run block only gets run when you are actually in the Spork DRB
environment, this ensures that you aren't unnecessarily re-loading your classes
outside of Spork.  The ActiveSupport::Dependencies.clear line re-loads all your
models and controllers. The next line re-instantiates your observers. Finally
the Sham::Config.activate! line reloads Sham so that you can easily create valid
model objects. TDD/BDD in real-time is possible again!
