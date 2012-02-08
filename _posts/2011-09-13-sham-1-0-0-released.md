---
layout: post
title: Sham 1.0.0 Released!
---

Today I released a new version of the
[Sham gem (version 1.0.0)](https://rubygems.org/gems/sham). This new version is
not backwards compatible, hence the major version release. I highly recommend
upgrading - this new version has some cool goodies included in it.

* Sham::Config.activate! is faster - much faster.
* Shams can be included individually. You no longer need to activate all
shams, you can simply include a file with a Sham definition in it and that Sham
will be available to you. You can also define Shams inline in your specs and
tests.
* The Sham definition syntax has changed, you can read more about this in the
[documentation](https://github.com/panthomakos/sham/blob/master/README.markdown).
* There are a few more helper functions, namely the #empty function which allows
you to create a simple empty Sham.

Here's a quick sample using the new definition syntax:

{% highlight ruby %}
Sham.config(Item){ |c| c.empty }

Sham.config(Item, :small) do |c|
  c.attributes do
    { :weight => 10.0 }
  end
end

Sham.config(Item, :large) do |c|
  c.attributes do
    { :weight => 100.0 }
  end
end

Item.sham!
Item.sham!(:small, :quantity => 100)
Item.sham!(:large, :build, :quantity => 0)
{% endhighlight %}

Enjoy!
