---
layout: post
title: 'The & Operator in Ruby'
categories: [Tutorial, Under the Hood]
tags: [ruby]
---

I'm going to assume that you are already familiar with the double ampersand
operator in Ruby - the logical AND. This post is going to focus on all uses of
the single ampersand operator. & can be quite confusing because it has a
different meaning depending on the context in which it's used. In fact, both
unary (`&object`) and binary (`object & object`) operations have meaning in
Ruby. To understand these let's look at the uses of & in core Ruby.

### The Binary Ampersand

In Ruby 1.9.3 there are three uses for the binary ampersand operator.

#### Bitwise AND

Bitwise AND is the binary bit-by-bit equivalent of boolean AND. So binary
`101 & 100 = 100` and binary `101 & 001 = 001`. & is defined as a bitwise AND
operator for `Bignum`, `Fixnum` and `Process::Status` and simply converts the
integer into a binary representation and performs a bitwise AND on that
representation. `Process::Status` converts the process status number to a
`Fixnum` and uses that to perform the operation.

{% highlight irb %}
irb(main):001:0> 14 & 13
=> 12
{% endhighlight %}

The result of this operation can be clearly seen by converting the numbers to
their binary representations.

{% highlight irb %}
irb(main):001:0> "#{14.to_s(2)} & #{13.to_s(2)} = #{12.to_s(2)}"
=> "1110 & 1101 = 1100"
{% endhighlight %}

#### Set Intersection

Possibly the simplest use of the binary & operator is in the `Array` class. &
is the set-intersection operator, which means the result is a collection of the
common elements in both arrays.

{% highlight irb %}
irb(main):001:0> [1,2,3] & [1,2,5,6]
=> [1, 2]
{% endhighlight %}

#### Boolean AND

On the `FalseClass`, `NilClass` and `TrueClass` binary & is the equivalent of
the boolean AND. Keep in mind this does not work like the && operator, since it
is only defined on these three classes.

{% highlight ruby %}
irb(main):001:0> false & true
=> false
irb(main):002:0> nil & true
=> false
irb(main):003:0> true & Object.new
=> true
irb(main):004:0> Object.new & true
=> NoMethodError: undefined method `&' for #<Object:0x007f9e7ac96420>
{% endhighlight %}

#### Custom Definitions

When the binary ampersand operator is invoked (`First & Second`) Ruby executes
the definition from the first object (`First#&`). You can write your own binary
ampersand method easily. When I do, I try to keep the first two core Ruby uses
in mind - bitwise AND and set intersection. I don't like using logical AND
because it is already covered by the `&&` operator and is only defined on
`true`, `false` and `nil`, which are special classes. Here is a custom example
that works like set intersection.

{% highlight ruby %}
DNA = Struct.new(:chain) do
  def triples
    chain.split(//).each_slice(3).to_a
  end

  def &(dna)
    (triples & dna.triples).map{ |triple| triple.join }
  end
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> DNA.new('AGGTTACCA') & DNA.new('TTAAGGCCC')
=> ["AGG", "TTA"]
{% endhighlight %}

### The Unary &

The Unary & is a little more complex. It is almost the equivalent of calling
`#to_proc` on the object, but not quite. To understand it let's go over some
background first. In Ruby you have two kinds of code blocks, Blocks and Procs.
The two are very closely related but have some important differences. You can
define and reference Procs and assign them to variables. Blocks are always
related to a method call and can't be defined outside of that context. The way
you tell them apart is that Procs are always preceded by `Proc.new`, `proc`,
`lambda` or `->()` when they are defined.

{% highlight ruby %}
# A block that is passed to the each function on the [1,2,3] Array.
[1,2,3].each do |x|
  puts x
end

# A proc assigned to a variable.
k = Proc.new{ |x| puts x }
{% endhighlight %}

All methods have one and only one implicit Block argument, whether you use it or
not. You can access it by calling `yield` in the method body.

{% highlight ruby %}
def two
  yield(2)
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> two{ |x| x*2 }
=> 4
{% endhighlight %}

Blocks are pretty useless outside of function calls though, for instance you
can't just define one.

{% highlight irb %}
irb(main):001:0> { |x| x*2 }
=> SyntaxError: syntax error, unexpected '|', expecting '}'
{% endhighlight %}

But you can define and reference Procs.

{% highlight irb %}
irb(main):001:0> Proc.new{ |x| x*2 }
=> #<Proc:0x007f9e7ab766a8@(pry):30>
{% endhighlight %}

Procs fall into two categories. Procs that are `lambda?`, lambda procs, and
Procs that aren't, simple procs. Lambdas are defined using `lambda` or `->()`
and whereas simple procs are defined using `Proc.new` or `proc`.

{% highlight irb %}
irb(main):001:0> lambda{ |x| x*2 }
=> #<Proc:0x007f9e8a8c3f40@(pry):34 (lambda)>
irb(main):002:0> ->(x){ x*2 }
=> #<Proc:0x007f9e7a896ed8@(pry):35 (lambda)>
irb(main):003:0> Proc.new{ |x| x*2 }
=> #<Proc:0x007f9e7a8f95d8@(pry):36>
irb(main):004:0> proc{ |x| x*2 }
=> #<Proc:0x007f9e7a950e28@(pry):37>
{% endhighlight %}

I'm not going to delve into the details of lambdas and simple procs, but there
are two basic differences and it's important to know that they exist. The first
is that lambdas are strict argument checkers, like methods, they can throw an
`ArgumentError` exception. Simple procs will just ignore incorrect, extra or
fewer argument combinations. The second is that lambdas act like methods
regarding their return status - they can return values just like methods. When
you try to return a value from a simple proc you end up with a `LocalJumpError`.

Now back to our unary ampersand operator. Since both Blocks and Procs are
useful, it's convenient to be able to switch between them - enter &.

`&object` is evaluated in the following way:

* if object is a block, it converts the block into a simple proc.
* if object is a Proc, it converts the object into a block while preserving the
`lambda?` status of the object.
* if object is not a Proc, it first calls #to_proc on the object and then
converts it into a block.

Let's examine each of these steps individually.

#### If object is a block, it converts the block to a simple proc.

The simplest example of this is when we want to have access to the block we pass
to a method, instead of just calling yield. To do this we need to convert the
block into a proc.

{% highlight ruby %}
def describe(&block)
  "The block that was passed has parameters: #{block.parameters}"
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> describe{ |a,b| }
=> "The block that was passed has parameters: [[:opt, :a], [:opt, :b]]"
irb(main):002:0> describe do |*args|
irb(main):003:0> end
=> "The block that was passed has parameters: [[:rest, :args]]"
{% endhighlight %}

#### If object is a Proc, it converts the object into a block while preserving
the `lambda?` status of the object.

This is an extremely useful case of the & operator. For instance, we know that
`Array#map` takes a block, but say we have a proc that we want to re-use in
multiple map calls.

{% highlight irb %}
irb(main):001:0> multiply = lambda{ |x| x*2 }

irb(main):002:0> [1,2,3].map(&multiply)
=> [2, 4, 6]
irb(main):003:0> [4,5,6].map(&multiply)
=> [8, 10, 12]
{% endhighlight %}

Keep in mind that the operator also preserves the `lambda?` status of the
original block. That's kind of neat because it means we are able to pass lambdas
(not simple procs) as blocks. That means we can impose strict argument checking
in our blocks and we can have them return values using the `return` keyword. The
only exception to this preservation is methods, which are always lambdas
regardless of how they are defined.

{% highlight ruby %}
def describe(&block)
  "Calling lambda? on the block results in #{block.lambda?}."
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> describe(&lambda{})
=> "Calling lambda? on the block results in true."
irb(main):002:0> describe(&proc{})
=> "Calling lambda? on the block results in false."
{% endhighlight %}
{% highlight ruby %}
class Container
  define_method(:m, &proc{})
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> describe(&Container.new.method(:m).to_proc)
=> "Calling lambda? on the block results in true."
{% endhighlight %}

#### If object is not a Proc, it first calls `#to_proc` on the object and then converts it into a block.

This is where the magic really happens because it makes passing objects to
functions in the place of blocks very simple. The most common case of this is
probably calling into `Array#map` with a symbol.

{% highlight irb %}
irb(main):001:0> ["1", "2", "3"].map(&:to_i)
=> [1, 2, 3]
{% endhighlight %}

This works because calling `Symbol#to_proc` returns a proc that responds to the
symbol's method. So the `:to_i` symbol is first converted to a proc, and then to
a block. This is kind of cool because we can create our own `to_proc` methods.

{% highlight ruby %}
class Display
  def self.to_proc
    lambda{ |x| puts(x) }
  end
end

class FancyDisplay
  def self.to_proc
    lambda{ |x| puts("** #{x} **") }
  end
end
{% endhighlight %}
{% highlight irb %}
irb(main):001:0> greetings = ["Hi", "Hello", "Welcome"]

irb(main):002:0> greetings.map(&Display)
Hi
Hello
Welcome
=> [nil, nil, nil]

irb(main):003:0> greetings.map(&FancyDisplay)
** Hi **
** Hello **
** Welcome **
=> [nil, nil, nil]
{% endhighlight %}

Hopefully this post has clarified the binary and unary ampersand operator in
Ruby.  It's a fun operator. In the binary form it can provide us with a
shorthand way of doing bitwise AND and set intersection like operations. In the
unary form it provides us with some powerful functionality for converting blocks
and procs.
