---
layout: post
title: Dynamic Rescue Clause in Ruby
categories: [Review, Tutorial]
tags: [jruby, ruby, exceptions]
---

I recently finished [Avdi Grimm](http://about.avdi.org/)'s
[Exceptional Ruby](http://exceptionalruby.com/). It's definitely a great
introduction to Ruby exceptions and how you can begin using and handling them in
your own projects and libraries.

While working through an example in his book I realized that Ruby 1.9.2 actually
greatly improves the rescue clause functionality. For instance, in Ruby 1.8.7
you can not use a proc to handle exceptions.

{% highlight ruby %}
#!/usr/bin/env ruby

def match_message(regexp)
  lambda{ |error| regexp === error.message }
end

begin
  raise StandardError, "Error message about a socket."
rescue match_message(/socket/) => error
  puts "Error #{error} matches /socket/; ignored."
end
{% endhighlight %}

Running this code in Ruby 1.8.7 produces a TypeError.

    :9: class or module required for rescue clause (TypeError)

Running this code in Ruby 1.9.2 results in a properly handled exception.

    Error StandardError matches /socket/; ignored.

To get this code to work in Ruby 1.8.7 you have to create a new module and
modify the === function on that module to handle the exception.

{% highlight ruby %}
#!/usr/bin/env ruby

def match_message(regexp)
  m = Module.new
  (class << m; self; end).instance_eval do
    define_method(:===) do |error|
      regexp === error.message
    end
  end
  m
end

begin
  raise StandardError, "Error message about a socket."
rescue match_message(/socket/) => error
  puts "Error #{error} matches /socket/; ignored."
end
{% endhighlight %}

Fortunately Ruby 1.9.2 entirely removes the module/class requirement for rescue
clauses which results in a
[27 times faster](https://gist.github.com/1230515#file_dynamic.bench.rb)
alternative when using a proc over the module metaprogramming approach. I
personally have not used this style of dynamic error handler in my code, I
prefer to subclass my exceptions, but if you are using another developer's
library this might be your only option to safely catch multiple exceptions.

For the jRuby fans out there, this is a syntax which jRuby does not support.
Even if you are running jRuby with the JRUBY_OPTS="--1.9" you'll have to use the
1.8.7 code.

You can view a gist of the examples in this post
[here](https://gist.github.com/1230515).
