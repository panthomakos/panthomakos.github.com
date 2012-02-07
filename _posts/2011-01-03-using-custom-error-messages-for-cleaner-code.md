---
layout: post
title: Using Custom Error Messages for Cleaner Code
category: Tutorial
tags: [refactoring, ruby, rails]
---

Either because you dread 500 Error Messages or you're on some kind of JSON kick
you find yourself writing this style of ruby code:

{% highlight ruby %}
class Group
  def join user
    if !self.permitted?(user)
      {
        :success => false,
        :message => "You don't have the proper permissions to join this group."
      }
    elsif self.member?(user)
      {
        :success => false,
        :message => "You are already a member of this group."
      }
    else
      self.members.create :user => user
      { :success => true, :message => "Welcome to the Group!" }
    end
  end
end

class GroupsController < ApplicationController
  def join
    @group = Group.find(params[:id])
    response = @group.join(current_user)
    if response[:success]
      flash[:notice] = response[:message]
      redirect_to @group
    else
      flash[:error] = response[:message]
      render :action => 'request'
    end
  end
end
{% endhighlight %}

Why?  Well because you don't want to risk throwing a Ruby Exception when it
could cause a 500 Error Message for a user, and you want to be able to control
error messages from your Model.  For example, the join method could return any
one of the following responses:

{% highlight ruby %}
{ :success => false, :message => "You are already a member of this group." }
{
  :success => false,
  :message => "You don't have the proper permissions to join this group."
}
{ :success => true, :message => "Welcome to the Group!." }
{% endhighlight %}

So what's wrong with that?  Well, there are a couple of things.

You introduced conditionals into your model and controller that are entirely
unnecessary.  Conditionals are not the worst thing in the world, but they create
divergent code paths, they make your code harder to understand and harder to
test.  If you can avoid them, it's easier for the next programmer to figure out
what you had in mind.

You created an additional dependency between your model and controller code.
Dependencies lead to bad things.  The more modular your code is the easier it is
to test and the more versatile it is.  It's easier to use modular code in
background jobs and other objects and functions.

Thirdly, and most importantly, you are replicating a perfectly good programming
concept known as Exceptions.  Exceptions are a program's way of saying:
something went wrong - something is not as it should be.  Any they are great to
use because they are easy to read and they break the normal code flow, which is
what you would expect if something went wrong.

How do we fix this code?  Simple: we use Exceptions.

<ol>
<li>
Create the custom exceptions.
{% highlight ruby %}
class Group
  module Error
    class Standard < StandardError; end

    class AlreadyAMember < Standard
      def message
        "You are already a member of this group."
      end
    end

    class NotPermittedToJoin < Standard
      def message
        "You don't have the proper permissions to join this group."
      end
    end
  end
end
{% endhighlight %}
</li>
<li>
Raise the exceptions.
{% highlight ruby %}
class Group
  def join user
    raise Error::NotPermittedToJoin unless self.permitted?(user)
    raise Error::AlreadyAMember if self.member?(user)
    self.members.create :user => user
  end
end
{% endhighlight %}
</li>
<li>
Catch the exceptions.
{% highlight ruby %}
class GroupsController < ApplicationController
  def join
    @group = Group.find(params[:id])
    @group.join current_user
    flash[:notice] = "Welcome to the Group!"
    redirect_to @group
  rescue Group::Error::Standard => exception
    flash[:error] = exception.message
    render :action => 'request'
  end
end
{% endhighlight %}
</li>
</ol>

A gist of this code can be found [here](https://gist.github.com/1230673).
