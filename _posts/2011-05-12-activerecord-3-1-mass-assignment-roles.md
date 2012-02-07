---
layout: post
title: ActiveRecord 3.1 - Mass-Assignment Roles
categories: [Reviews]
tags:
- active-record
- complaints
- rails-3.1
- ruby on rails
---

[Rails 3.1 introduces the concept of mass-assignment roles][ma] which allow you
to specify accessible attributes for a specific permission or role. For
instance, if you wanted your users to be able to edit their names, but not their
permissions you could limit their accessible attributes but allow administrators
to edit permissions:

[ma]: https://gist.github.com/958283

{% highlight ruby %}
class User < ActiveRecord::Base
  attr_accessible :name
  attr_accessible :name, :permission, :as => :admin
end
{% endhighlight %}

While this syntax is awesome the AR#new, AR#create and AR#update_attributes
syntax is horrible:

{% highlight ruby %}User.create(params[:user], :as => :admin){% endhighlight %}

It seems like the same primitive syntax that AR#find used in Rails 2.x:

{% highlight ruby %}
User.find(:all, :conditions => ['name = ?', params[:name]])
{% endhighlight %}

That was replaced with elegant chaining in Rails 3.0:

{% highlight ruby %}User.where('name = ?', name).all{% endhighlight %}

Why not do the same with mass-assignment roles?

{% highlight ruby %}User.as(:admin).create(params[:user]){% endhighlight %}

or

{% highlight ruby %}User.as(:admin).attributes(params[:user]).create{% endhighlight %}

I can see the conflict this might cause with scopes, but maybe there's an
elegant balance so that we aren't passing multiple hash parameters to AR#new,
AR#create and AR#update_attributes.
