---
layout: post
title: 'after_create bug in Rails 3'
categories: [Review]
tags:
  - active record
  - rails
  - ruby
---

I ran into this callback ordering issue while working on a Rails project
recently and decided to investigate.  After writing some test rails apps I am
pretty confident this is a bug in Rails 3.0.x.  If you believe otherwise, please
leave a comment or explanation.

Taking a step back from the actual code, in an ORM (Object Relational Model) I
would expect that callbacks from objects are consistent in their ordering.  That
is, if I have a callback that fires when an object has been saved to the
database, that callback will fire at the same time (after the object has been
saved to the database) regardless of any dependent objects or where the callback
has been declared.  Here's some code to demonstrate this:

{% highlight ruby %}
class User
  after_create { puts 'after create user' }
  has_many :posts
end

class Post
  after_create { puts 'after create post' }
  belongs_to :user
end

user = User.new
user.posts.build
user.save
=> 'after commit user'
=> 'after commit post'
{% endhighlight %}

Simple, right? Almost. The problem in Rails is that the after_create callback
doesn't exactly work that way. When it fires is actually dependent on the order
in which it was defined. What's more - this isn't actually the case with the
after destroy callback. Here's some code to demonstrate the issue:

{% highlight ruby %}
class User
  after_create { puts 'User - first after create' }
  after_destroy { puts 'User - first after destroy' }

  has_many :posts, :dependent => :destroy

  after_create { puts 'User - second after create' }
  after_destroy { puts 'User - second after destroy' }
end

class Post
  after_create { puts 'Post - first after create' }
  after_destroy { puts 'Post - first after destroy' }

  belongs_to :user

  after_create { puts 'Post - second after create' }
  after_destroy { puts 'Post - second after destroy' }
end

user = User.new
user.posts.build
user.save
=> User - first after create
=> Post - first after create
=> Post - second after create
=> User - second after create

user.destroy
=> Post - first after destroy
=> Post - second after destroy
=> User - first after destroy
=> User - second after destroy
{% endhighlight %}

Take a second and look at that - do you see what's wrong? The
`User#after_create` callbacks are called at opposite ends of the Post#after
create callbacks, but the User#after_destroy callbacks are called at the end.
What does this mean? Well basically that after_create is not actually called
after the record is created (and saved to the database) - it's called "after the
record is created and any dependent relations defined before it are also
created". Now that just seems wrong to me. Especially when the after_destroy
callback behaves in a different way.

It seems that the flow should work like this:

1. The user is saved.
2. The `User#after_create` callbacks are executed.
3. The post is saved
4. The `Post#after_create` callbacks are executed.

Especially when the opposite is true for `after_destroy`:

1. The post is destroyed.
2. The `Post#after_destroy` callbacks are executed.
3. The user is destroyed.
4. The `User#after_destroy` callbacks are executed.

The problem right now is that the `User#after_create` callbacks are being called
in two places - once after the user is created (if they are defined before the
`has_many :posts` declaration) and once after the `Post#after_create` callbacks
are executed (if they are defined after the has_many :posts declaration).

If you're into TDD, you would check that the first case actually produces this
result:

    => User - first after create
    => User - second after create
    => Post - first after create
    => Post - second after create

So why is this a problem? Why not just define all your callbacks before your
relations? The first problem with this is that it is inconsistent - after
destroy and after_create should behave in the same manner. The second problem is
that your code might fail and it's not obvious why.

{% highlight ruby %}
class User
  include Poster

  after_create :do_something
end

module Poster
  def self.included klass
    klass.class_eval do
      has_many :posts, :dependent => :destroy
    end
  end
end
{% endhighlight %}

While this example is trivial it illustrates how easy it is to mix up the order
or definitions in Ruby. Unbeknownst to us, simply looking at the User class,
this would actually cause our :do_something callback to be executed in the wrong
order.
