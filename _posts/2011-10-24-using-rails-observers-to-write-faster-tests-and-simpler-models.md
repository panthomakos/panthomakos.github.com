---
layout: post
title: Using Rails Observers to write Faster Tests and Simpler Models
categories: [Best Practices, Refactoring]
tags: [performance, ruby, rails, tdd]
---

[Rails observers](http://api.rubyonrails.org/classes/ActiveRecord/Observer.html)
provide a great way to organize your code. If you follow the
skinny-controller-fat-model practice in Rails then you may have already crossed
the line from comfortably-plump models into massively-obese models. In fact it's
a common misconception that your "models" need to map directly to database
tables and it's quite easy to get carried away adding methods, associations,
scopes, validations and callbacks to them. Do not fear - one of the benefits of
writing Agile Object-Oriented Ruby is that you can easily simplify this mess
with a little refactoring. I'm only going to touch on refactoring callbacks
today, but a lot of what I cover here can be implemented for other methods as
well.

### What's Wrong with This Callback?

Let's consider the following seemingly innocent code. Assume for the time being
that in production our emails are actually delivered in the background.

{% highlight ruby %}
class User
  after_create :send_welcome_email

  protected

  def send_welcome_email
    if purchased_membership?
      GreetingMailer.welcome_and_thanks_email(self).deliver
    else
      GreetingMailer.welcome_email(self).deliver
    end
  end
end
{% endhighlight %}

Does this callback really belong on the `User` class? In some ways it does, it's
definitely entirely dependent on the user's state and does a pretty good job of
delegating to the `GreetingMailer` instead of trying to handle the mail
implementation itself. But it does kind of break the "each object should only
deal with one or a few things" OOP pattern. On a more academic level this code
doesn't really have much to do with the user domain model. It's only really
responsible for sending a welcome email when something is created.

### Let's Refactor It

Let's rid the `User` model of this coupling by refactoring the code into an
observer. Our `WelcomeEmailObserver` will have a single responsibility, namely
to "send the appropriate welcome email once something is created".

{% highlight ruby %}
# app/[models|observers]/welcome_email_observer.rb
class WelcomeEmailObserver < ActiveRecord::Observer
  observe :user

  def after_create(user)
    if user.purchased_membership?
      GreetingMailer.welcome_and_thanks_email(user).deliver
    else
      GreetingMailer.welcome_email(user).deliver
    end
  end
end

# app/models/user.rb
class User
end

# config/application.rb
class Application < Rails::Application
  config.active_record.observers = :welcome_email_observer
end
{% endhighlight %}

Looks good. A couple of things to notice. First, the `User` model is simpler.
It is no longer coupled to the `GreetingMailer`. In fact, removing or modifying
the `GreetingMailer` does not require us to touch the `User` model. A
side-effect of this is also that the `WelcomeEmailObserver` is reusable. It is
no longer strictly coupled to the `User`. Secondly, we aren't messing with
[The Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter), the
`WelcomeEmailObserver` is only referencing its own parameters and first level
attributes of those parameters.

### Fixing Our Tests

Now that we've moved things around, let's take a look at our tests. First we'll
ensure that they pass and that we didn't break anything.

{% highlight ruby %}
describe User do
  context 'welcome email' do
    let(:mail){ stub(:deliver) }

    it 'should send the simple welcome email if not purchased' do
      WelcomeMailer
        .should_receive(:welcome_email)
        .and_return(mail)
      User.any_instance.stub(:purchased_membership?).and_return(false)
      User.sham!
    end

    it 'should send the welcome and thanks email if purchased' do
      WelcomeMailer
        .should_receive(:welcome_and_thanks_email)
        .and_return(mail)
      User.any_instance.stub(:purchased_membership?).and_return(true)
      User.sham!
    end
  end
end
{% endhighlight %}

Once we've verified that the code works, we'll move the tests from our
`user_spec` into our `welcome_email_observer_spec`. After all we're no longer
really testing the `User`.

{% highlight ruby %}
describe WelcomeEmailObserver do
  let(:mail){ stub(:deliver) }

  it 'should send the simple welcome email if not purchased' do
    WelcomeMailer
      .should_receive(:welcome_email)
      .and_return(mail)
    WelcomeEmailObserver.new
      .after_create(stub(:purchased_membership? => false))
  end

  it 'should send the welcome and thanks email if purchased' do
    WelcomeMailer
      .should_receive(:welcome_and_thanks_email)
      .and_return(mail)
    WelcomeEmailObserver.new
      .after_create(stub(:purchased_membership? => true))
  end
end
{% endhighlight %}

That's a great first step, we've definitely simplified our test and are no
longer coupled to the `User` class.

### Reducing Test Dependencies

While it's not immediately obvious, our first implementation actually strongly
coupled all of our tests that create a `User` to the `GreetingMailer`. Any time
we created a user we inadvertently also delivered a welcome email. Aside from
creating a dependency between our tests and the success of `GreetingMailer`
methods we've also incurred a great performance penalty, namely the code related
to constructing and rendering the email.

While the rendering part may not apply to all of our tests it's easy to deduce
that almost any callback will impose a similar and undesirable overhead. This
simple performance penalty is really the main reason that so many Rails test
suites take so long to run.

Let's take a look at a solution to this overhead and how we can simplify our
test.

### Improving Test Performance

First, we'll install the
[no_peeping_toms](https://rubygems.org/gems/no_peeping_toms) gem. This gem
allows us to turn off ActiveRecord observers on a case-by-case basis. I prefer
to implement it by adding the following to my spec helper file.

{% highlight ruby %}
RSpec.configure do |config|
  config.before(:suite) do
    ActiveRecord::Observer.disable_observers
  end
end
{% endhighlight %}

If you already have observers that are being tested, this change will most
likely causes a lot of your tests to fail until you enable the related observers
on individual tests. But alone this change can result in a very noticeable
performance boost.

The one thing we might be missing from this refactoring is a good integration
test. While it can be argued that testing observer integration is really just
testing Rails, we can make an argument that an integration test is actually
simply testing that our observer is in fact at least observing the `User` model.
In other words that our observer contains the line `observer :user`.  So, if we
are compelled to do so, we can also add an integration test to ensure this.

{% highlight ruby %}
describe User do
  it 'should send a welcome email on create' do
    WelcomeEmailObserver.any_instance.should_receive(:after_create).once
    ActiveRecord::Observer.with_observers(:welcome_email_observer) do
      User.sham!
    end
  end
end
{% endhighlight %}

Hopefully this post has enlightened or convinced you of the use of observers and
given you some insight into testing them and improving the performance and
reliability of the remainder of your tests. I would like to continue writing
some more posts about refactoring and testing improvements, so if you have any
suggestions please email me or leave a comment.
