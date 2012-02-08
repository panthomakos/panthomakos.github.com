---
layout: post
title: Scopes in Rails 3
category: Tutorial
tags:
  - ruby
  - active record
  - rails
  - refactoring
---

Scopes have become much more useful in Ruby on Rails 3 with the adoption of
[Arel](https://github.com/rails/arel) into ActiveRecord. The first and most
obvious benefit related to scopes is that everything is scoped! That means you
no longer have to use find(...) on an ActiveRecord model and have a query
immediately be executed. You can actually build queries up and only execute
them when you use them. For example:

In Rails 2.x the find method worked like this:

{% highlight ruby %}
User.find(
  :all,
  :joins => :profile,
  :conditions => ['profile.age = ?', 33])
{% endhighlight %}

This would then execute the query and find all users with age 33. In Rails 3.x
this is much simpler:

{% highlight ruby %}
User.joins(:profile).where('profile.age = ?', 33)
{% endhighlight %}

Not only is this more readable since you don't have to put your entire query
into a single function call, but the main difference between these two commands
is that the second doesn't execute a query. You actually have to call .all,
.count, .each or .first to get the query to execute. What's great about that?
Well, it means you can chain conditions together before you execute them:

{% highlight ruby %}
query = User.joins(:profile).where('profile.age = ?', 33)
query.where('users.name = ?', name) unless name.nil?
query.where('profile.email = ?', email) unless email.nil?
query.all
{% endhighlight %}

This really shines when you combine it with scopes. For instance, if we want to
simplify the above code, we can do the following:

{% highlight ruby %}
class User
  scope :by_age, lambda do |age|
    joins(:profile).where('profile.age = ?', age) unless age.nil?
  end
  scope :by_name, lambda{ |name| where(name: name) unless name.nil? }
  scope :by_email, lambda do |email|
    joins(:profile).where('profile.email = ?', email) unless email.nil?
  end
end

User.by_age(33).by_name(params[:name]).by_email(params[:email]).all
{% endhighlight %}

Isn't that just awesome!? Not only is the code easier to read and understand,
but it's re-usable and scoped! Here's one more example:

{% highlight ruby %}
class Tag
  belongs_to :post
end

class Post
  has_many :tags
  belongs_to :user

  scope :tagged, lambda do |tag|
    joins(:tags).where('tags.name = ?', tag).group('posts.id')
  end
end

class User
  has_many :posts
end

# How many of my posts are tagged with 'ruby-on-rails'?
User.where('users.id = ?', 232423).posts.tagged('ruby-on-rails').count
{% endhighlight %}
