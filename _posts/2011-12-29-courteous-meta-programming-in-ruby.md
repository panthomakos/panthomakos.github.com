---
layout: post
title: Courteous Meta-Programming in Ruby
categories: [Best Practices]
tags: [metaprogramming, ruby]
---

Meta-programming is often kind of mean. It has a tendency to be hard to
understand and it can be quite destructive - causing methods to be modified or
overwritten in non-obvious ways. I decided to write a post about a more
courteous form of meta-programming. I picked up the technique from
[a blog post about method_missing and respond_to?][post]. I realized that
although I am comfortable with meta-programming, other developers might not feel
the same way. So, here's a simple pattern that can make your meta-programming
more readable and less destructive.

[post]: http://avdi.org/devblog/2011/12/07/defining-method_missing-and-respond_to-at-the-same-time/

I want to create a module named `Greetings` so that when I extend my classes
with it I can declaratively add greetings to them.

{% highlight ruby %}
class FrenchAndEnglish
  extend Greetings
  greeting "Bonjour!"
  greeting "Hello!"
end
{% endhighlight %}
{% highlight irb%}
irb(main):001:0> FrenchAndEnglish.new.greet
Bonjour!
Hello!
=> nil
{% endhighlight %}

There are a couple of ways to accomplish this with Ruby. But before we jump into
code, let's add these constraints to the system. After all, we are trying to
write courteous code.

* **Readability** - the code should be easy to read and understand with minimal
knowledge of meta-programming.
* **Super Support** - the `greet` method should support the use of `super` to
delegate to parent definitions.
* **Non Destructive** - the extending class should be able to define it's own
`greet` method.

For instance, consider the following use of the `Greetings` module. It supports
the use of `super` to delegate to previous implementations of `greet` and it
allows the base class to define it's own `greet` method.

{% highlight ruby %}
class TriLingualWithEnglish
  extend Greetings
  greeting 'Hello!'
  greeting 'Howdy!'

  def initialize(message01, message02)
    @message01 = message01
    @message02 = message02
  end

  def greet
    super rescue nil
    puts "#{@message01} and #{@message02}!"
  end
end
{% endhighlight %}

{% highlight irb %}
irb(main):001:0> TriLingualWithEnglish.new('Bonjour', 'Kalimera').greet
Hello!
Howdy!
Bonjour and Kalimera!
=> nil
{% endhighlight %}

Let's look at the most common meta-programming approaches to solving this
problem.

* **eval** - using `instance_eval` or `class_eval` might be acceptable, but
these options are generally destructive. They overwrite methods on the base
class and this breaks our non-destructive requirement. If we were able to solve
this problem by using alias method chains then we would definitely be entering
the territory of un-readable code. Also method chains would not allow us to use
`super` to call previous definitions of `greet`.
* **Tracking Class Variable** - a variable defined on the class level that is
responsible for keeping a list of greetings. This has lots of problems, for one
it does not support `super`, but it also greatly restricts our flexibility.  We
can no longer define an alternate `greet` method on the base class.

Since these two approaches don't fit our courteous meta-programming
requirements, here's the cool alternative that does.

{% highlight ruby %}
module Greetings
  def greeting(phrase)
    m = Module.new do
      define_method(:greet) do
        super rescue nil
        puts(phrase)
      end
    end

    include m
  end
end
{% endhighlight %}

By using the anonymous module `m`, and including it we are actually creating a
storage object in the class hierarcy for the method, instead of writing it on
the original class. This approach is non-destructive and allows us to maintain
a clean method chain that supports the use of `super`. The `super rescue nil`
line in our `TriLingualWithEnglish` class now calls into the anonymous module
and allows both greeting implementations to co-exist. The code is also very
readable since the programmer does not need to understand `class_eval` and
`instance_eval`. I would advocate this approach whenever possible since it
preserves the class hierarchy and avoids some of the less readable
meta-programming techniques.
