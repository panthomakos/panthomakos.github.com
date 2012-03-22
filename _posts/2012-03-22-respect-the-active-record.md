---
layout: post
title: Respect the Active Record
category: Best Practices
tags: [rails, active record]
---

Here's some bad Rails code I often come across:

{% highlight ruby %}
Author.where(public: true).where('score > ?', 50)
{% endhighlight %}

[The problem isn't that we're chaining][post]. Because we're only handling
`ActiveRecord::Relation` objects, we aren't increasing the complexity or
responsibility of our code by the mere act of chaining. Chaining is a useful
programming pattern. However, we are increasing the complexity of our code by
making assumptions about the internal workings of `Author` - we're breaking
encapsulation. Even if you feel that `ActiveRecord` makes this easy to do, it
doesn't mean you should do it.

[post]: /2012/02/27/understanding-the-law-of-demeter/

We shouldn't know that authors have internal `score` and `public` attributes.
Keep in mind that this is different from being able to call `Author#score` and
`Author#public`, because in the former case, we actually "know" a lot more about
`Author` and we make some unhealthy assumptions.

In particular we make the assumption that these two statements are logically
equivalent:

{% highlight ruby %}
Author.all.select{ |u| u.score == 50 }

Author.where(score: 50).all
{% endhighlight %}

In short, this code is asking `Author` to perform an operation based on an
assumed internal state. We are asking - not telling. That's not good.

Telling `Author` to give us this data looks very different:

{% highlight ruby %}
Author.with_public_account.with_score_above(50)
{% endhighlight %}

Having this kind of interface for our objects benefits us in a couple of
really awesome ways.

### Readability

Using intentionally crafted public interfaces for your objects makes them easier
to read and more descriptive. This is usually a by-product of the simple fact
that you are naming your methods instead of using what is already there. Keep in
mind that ORMs were created as a general infrastructure for many different kinds
of applications. There is no need to expose that infrastructure to your own
application. It makes more sense to write `Author.with_public_account` than it
does to write `Author.where(public: true)`. The intent is clearly communicated.

### Searchability

Searching for `where.*score` in a code base can be tedious and might not result
in catching all the possible use cases. Code structured like this can easily be
missed:

{% highlight ruby %}
conditions = {score: 50}
Author.where(conditions)
{% endhighlight %}

By contrast, searching for `with_score_above` is much more likely to yield
usable and accurate results. Obviously having a full test suite helps alleviate
this kind of a refactoring, but that doesn't mean you should make it difficult
for others to track down your handiwork.

### Changeability

Have a more explicit interface makes it easier to change our ORM from
`ActiveRecord` if we ever choose to. It makes it easier to make small attribute
level changes, like renaming score. That's because we now only need to modify
the `Author` object, not every piece of code that filters authors on score. You
could imagine a future implementation that looks something like this:

{% highlight ruby %}
module AuthorRelation
  def initialize(authors)
    @authors = authors
  end

  def with_public_account
    @authors = @authors.select{ |a| a.public? && a.paid? }
    self
  end

  def with_score_above(score)
    @authors = @authors.select{ |a| a.cached_score > score }
    self
  end
end
{% endhighlight %}

If you're thinking - why would I ever want to change something like that? Then
you've missed the point. This isn't a pre-optimization, this is a philosophy for
creating persistence objects that behave like simple Ruby objects, without
exception. It's a slippery slope to assume that some objects just aren't going
to change and that kind of thinking causes headaches for the rest of your code
and for other developers working with code you've written.

What if we didn't change the entire object; what if we simply denormalized and
indexed a flag that combined `score` and `public`? Our carefully crafted public
interface would allow us to make use of this new column, whereas being tied to
the `where` way of doing things would mean having to modify a lot more code.

### Testability

It's easier to begin building an application without being tied to the
`ActiveRecord` way of doing things. This frees us up to write isolated tests and
hide `ActiveRecord` as a true implementation detail, rather than making it a
central part of our application. No object or collection of objects should
dominate your code. Some objects might be more or less connected, but that
doesn't mean they should be required to test one another. Isolation is a great
thing.

### Summary

Respect the `ActiveRecord`. Let it do what it does best, as a piece of
infrastructure and as a 3rd party library. Its way of doing things and its
method signatures should not infect the rest of your application. It does what
it does best when it's limited to and hidden inside of your `ActiveRecord::Base`
objects.
