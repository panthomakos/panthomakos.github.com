---
layout: post
title: Understanding The Law of Demeter
category: Best Practices
tags: [ruby]
---

The Law of Demeter is often stated as, "only talk to your immediate friends."
In Object Oriented languages that use a dot as the field identifier this is
often simplified to, "only use one dot." While this is an interesting heuristic
of sorts, it's a very poor rule to follow because it's almost never that simple.
This is often hard to describe to new programmers, who tend to seek a set of
rules to help them advance their skills. The reality is that there exists no
simple rule you can follow, it's really more of a sense. The best way I've been
able to come up with to summarize the Law of Demeter is, "don't operate on
objects that you might not understand."

To illustrate this, let's consider things from the perspective of the calling
object, `self` in Ruby. Say we have an expression `self.x.y`. Then we should be
less likely to tell the receiver, `x`, to do `y`, as it becomes harder to
understand `x` or as it becomes more likely for `x` to change.

In the simplest case the receiver is the caller. This case is equivalent to the
original and stricter Law of Demeter. So in the expression `self.x`, `self` is
the caller and the receiver. And since we have a high level of confidence in our
own methods, we can call `x`.

In the more general case, given `self.x.y.z`, if our level of confidence in `x`
is low, we should not be telling `x` to do `y`. And this carries forward - if,
from the perspective of `self`, we do not have high confidence in `y`, then we
should not be telling it to do `z`.

# A Ruby Example

{% highlight ruby %}
[1,2,3,4].map{ |k| k*2 }.join(',')
{% endhighlight %}

Using the original definition this code violates the law since the call to
`Array#join` is a case of talking to a non-immediate friend. `[1,2,3,4]` is
the immediate friend of `self` in this case. The call to `Array#map` is fine,
but the result of that operation is not an immediate friend, so we can't tell
it to do anything.

Fixing this code leads us down a rat-hole and is highly non-productive. It most
likely results in greatly increasing the complexity of an otherwise pretty
simple piece of code. In fact this simple example demonstrates that under a
strict version of the Law of Demeter we can't chain methods at all.

To solve this chaining problem, most people tend to introduce this caveat:
'your friend's friend is also your friend if she is of the same type as your
your friend'. In our previous example `Array#map` returns an array and we
started with an array, so we can act on the result of that operation. But we
can't act on the string that `Array#join` returns because we didn't start with a
string.

This caveat is not enough though. In some cases it's too restrictive and in
others it simply is not restrictive enough.

# An Example of Complex Code

{% highlight ruby %}
User
  .where(active: true)
  .map(&:id)
  .map{ |id| Group.where(owner_id: id) }
  .map(&:name)
  .grep('Rails')
{% endhighlight %}

This is only slightly a violation of the Law of Demeter, but it is definitely a
violation of the fuzzier version I introduced earlier. The issue here is really
the complexity of the code. There is a good chance that the code related to
either `User` or `Group` will change and it's not feasible to expect that we
will know when that happens. Additionally, this code assumes that we have
confidence in each operation in the chain, and that's not something I feel
comfortable with. Not because any one call in the chain is complex, but because
by the time we get to the call that deals with grepping for 'Rails', we have
already lost track of what we were doing. Take a look at a simplified version of
this code that addresses these issues.

{% highlight ruby %}
users = User.active

Group.owned_by_any_of(users).with_name_matching('Rails')
{% endhighlight %}

That's the beauty of the Law of Demeter. It advocates writing code like this
over the first example. It's not really about how many dots you are using, it's
about the responsibility of the objects in your code. How many objects is the
caller responsible for understanding and keeping track of? How much logic is the
caller encompassing? And is the logical path of the chain easy to follow?

# An Example of Unknown Output

{% highlight ruby %}
user.group.try(:name)
{% endhighlight %}

If you find yourself writing code like this, then you're in violation of the
Law of Demeter. Both versions. And this code should be simplified.

{% highlight ruby %}
user.group_name
{% endhighlight %}

Let's take a look at a slight variation to the first example.

{% highlight ruby %}
user.group.name
{% endhighlight %}

In this code there's a good chance that you're in the clear. In fact this code
is quite easy to understand, and if we actually do have confidence that `group`
exists and that we'll understand the output, it's perfectly fine to use this
expression.

# Summary

The Law of Demeter is not a law, and it's not as simple as counting the number
of dots in your statements. It's really about developing a sense for
constructing your code so that each object is confident in what it is telling
other objects to do, and what it is responsible for understanding. Hopefully
this post clarified the Law of Demeter for you.
