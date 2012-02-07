---
layout: post
title: Ruby Autoloading Explained
category: Under the Hood
tags: [rails, ruby]
---

In the ruby programming language a distinction is made between loading and
autoloading.  For those of you familiar with Rails, this will shed some light on
the change from `config.load_paths` to `config.autoload_paths` in Rails 3. The
difference lies in **when** the class is actually called/initialized.

Let's say I have defined a SpecialLibrary class:

{% highlight ruby %}
# In special_library.rb
class SpecialLibrary
  puts "Loading Special Library"
  ...
end
{% endhighlight %}

When I load or require the class, the entire class code is run:

{% highlight irb %}
irb(main):001:0> require 'special_library'
Loading Special Library
=> true
{% endhighlight %}

When I autoload the class, the class code is only run when I actually use the
class:

{% highlight irb %}
irb(main):001:0> autoload :SpecialLibrary, 'special_library'
=> nil
irb(main):002:0> SpecialLibrary.new
Loading Special Library
=> #
{% endhighlight %}

The reason `config.load_paths` was changed to `config.autoload_paths` in Rails 3
is because the paths defined here were being auto-loaded and not simply
required, so `config.autoload_paths` is more descriptive and accurately
describes the behavior.
