---
layout: post
title: Object Oriented Ruby and Rails in the Complex World of Gems
categories: [Best Practices]
tags:
  - active record
  - rspec
  - ruby
  - rails
---

Good Object-Oriented Programmers think of Objects and Responsibilities in terms
of a one-to-one relationship. One object, one responsibility. This concept
implies that applications should have a dichotomy between objects that are
responsible for business logic and objects that are responsible for integrating
with Gems. Another way to think of this is that your business logic should not
also be responsible for understanding a 3rd Party API. The knowledge of how to
interact with a gem shouldn't be duplicated in two different areas of your
application. Ideally, if you upgrade or replace a gem you should only have to
change one piece of code to make your application fully compatible with the new
gem version. That way your code is less brittle and easier to maintain.

### A Simple Example

Say you have a pagination gem that works by monkey-patching the Array object.

{% highlight ruby %}Array#paginate(per_page, page){% endhighlight %}

To wrap this functionality you could create the following object.

{% highlight ruby %}
class Paginator
  def self.paginate(array, per_page, page)
    array.paginate(per_page, page)
  end
end
{% endhighlight %}

When you upgrade that particular gem you only have one piece of code that you
need to change. And when an exception is raised, it is raised from within the
`Paginator` which is a clean and obvious way of notifying you that the API has
probably changed. If the author of that gem chooses to use named parameters in
a hash in the next gem version, you can simply modify your `Paginator` object.

{% highlight ruby %}
class Paginator
  def self.paginate(array, per_page, page)
    array.paginate(per_page: per_page, page: page)
  end
end
{% endhighlight %}

### The Two Solutions

At some point my objects need to interact with external libraries directly, but
I like to minimize where that happens.

The first way to achieve this is to use the
[Facade Pattern](http://en.wikipedia.org/wiki/Facade_pattern) and have your
classes interact with the facade object rather than the gem. The facade object
is responsible for abstracting the gem API and is customized to fit the needs of
your application.

The second way to achieve this is to use what I like to call the Isolation
Pattern. In the Isolation Pattern the objects that interact with the external
library meet two requirements. First, they are relegated to the single
responsibility of interacting with the gem. Second, they are kept in their own
module or directory so as to prevent any kind of mingling with your other
objects.

The two approaches achieve a similar end result - only one area of your code is
responsible for interaction with gem X. And that's the important take-away.
Let's take a look at three examples.

### RSpec

RSpec clearly falls into the Isolation Pattern solution. It is setup to operate
in the `spec/` directory with files named `*_spec.rb`. It should not creep into
any other application files. It's also a very well known API. Other developers
will find it easier to interact with RSpec directly than with a facade that you
have created.

### Facebook and Twitter

Facebook and Twitter authentication gems fall into the Facade Pattern solution.
They are highly specialized and don't have a common or popular interface. Most
developers won't need to interact with these gems on a daily basis, so a facade
makes sense because it simplifies an often complex API.

### ActiveRecord

ActiveRecord - now this one may seem complex because Rails developers tend to
put everything under the sun into their ActiveRecord classes, but it really
falls into the Isolation Pattern solution. I like to think of ActiveRecord
objects as Objects that have the Responsibility of persisting your domain data.
That's it. They sit in the `app/models` directory. They aren't responsible for
authenticating my Users or suggesting other Users for them to follow. It's
actually quite simple, here are my two ActiveRecord rules.

<ol>
<li>
Don't call ActiveRecord finders directly.

Finders, like `where`, are specific to ActiveRecord. They should not be called
outside of the ActiveRecord classes.

{% highlight ruby %}
class PostsController
  def index
    @posts = Post.where(:created_at.gte => Date.today-1.month)
  end
end
{% endhighlight %}

Should actually be

{% highlight ruby %}
class PostsController
  def index
    @posts = Post.in_the_last_month
  end
end
{% endhighlight %}

The reason for this is that if ActiveRecord changes its query interface as it
did between Rails 2 and Rails 3 then we shouldn't have to change our controllers
and application logic.
</li>
<li>
Don't put domain logic in your ActiveRecord classes.

{% highlight ruby %}
class User < ActiveRecord::Base
  def self.authenticate(username, password)
    user = where(username: username).first
    ...
  end
end
{% endhighlight %}

Should actually be

{% highlight ruby %}
class UserAuthenticator
  def self.authenticate(username, password)
    user = User.by_username(username)
    ...
  end
end
{% endhighlight %}

The reason for this is that user authentication is application logic. It should
not be dependent upon our persistence layer.
</li>
</ol>

In a small application this may seem cumbersome.  Why create so many extra
objects to handle such simple functionality? As your application grows and more
developers need to understand it and test it, using good
Object-Oriented-Programming patterns makes sense. It makes your life easier and
it makes coding more fun. There are few things worse than
large-multi-responsible-classes.  Massive classes take too long to understand
and force longer and more complex function names. By following One Object, One
Responsibility you can separate implementation logic from business logic and
make your system less brittle, easier to test and easier to scale.
