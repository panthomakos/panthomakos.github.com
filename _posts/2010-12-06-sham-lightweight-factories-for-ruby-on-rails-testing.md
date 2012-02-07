---
layout: post
title: Sham - Lightweight Factories for Ruby on Rails Testing
categories: [Tutorial]
tags: [cucumber, factories, rspec, sham, tdd]
---

In general test-driven development in Rails couldn't be easier, but while
writing your tests you'll eventually run into the case where you need to create
a valid model so that you can test some extended functionality. Take a look at
this RSpec test.

{% highlight ruby %}
describe Cart do
  it "should be able to calculate the total price" do
    user = User.create \
      :username => "username@example.com",
      :password => "password", 
      :password_confirmation => "password"
    one = Item.create :name => "Item One", :price => 10.00, :weight => 1.0
    two = Item.create :name => "Item Two", :price => 20.00, :weight => 2.0
    cart = Cart.create :user => user
    li1 = LineItem.create :cart => cart, :item => one, :quantity => 1
    li2 = LineItem.create :cart => cart, :item => two, :quantity => 1
    cart.price.should == li1.price+li2.price
  end
end
{% endhighlight %}

The problem with this approach is that if you modify the validations on `Item`,
`Cart` or `User` you'll end up going back and adjusting your test.  For example,
if you required that items had a quantity that was positive, you would have to
go back and add a quantity parameter every time you create an item -  major pain
in the ass.  Not to mention your tests start to become quite wordy.

{% highlight ruby %}
one = Item.create :name => "Item One",
        :price => 10.00,
        :weight => 1.0,
        :quantity => 10
{% endhighlight %}

Fortunately there are a couple of gems that are available for creating
factories. Factories are "minimally valid models" that can be reused to create
valid models.  I have experimented with a couple of options, like
[factory girl](https://github.com/thoughtbot/factory_girl) or
[machinist](https://github.com/notahat/machinist). But I ran into issues with
sequences (sequentially generated attributes that are used to avoid validation
errors) or I disliked the way the gems and factories were setup and used in my
tests.  This led me to create my own lightweight factory gem called
[sham](https://github.com/panthomakos/sham).

Here's how you can get started using Sham.

## 1. Install the Sham Gem.
{% highlight bash %}gem install sham{% endhighlight %}
Or add it to your Gemfile:
{% highlight ruby %}gem "sham"{% endhighlight %}
And re-install your Bundle:
{% highlight bash %}bundle install{% endhighlight %}


## 2a. If you are using [RSpec](http://rspec.info/) or [Test::Unit][test-unit], enable Sham in your test.rb file.

[test-unit]: http://www.ensta.fr/~diam/ruby/online/ruby-doc-stdlib/libdoc/test/unit/rdoc/classes/Test/Unit.html

{% highlight ruby %}
config.after_initialize do
  Sham::Config.activate!
end
{% endhighlight %}

## 2b. If you are using [Cucumber](http://cukes.info/), enable Sham in your features/support/env.rb file.</h2>

{% highlight ruby %}
require 'sham'
Sham::Config.activate!
{% endhighlight %}

## 3. Create factories with your default attributes.

{% highlight ruby %}
# in sham/item_sham.rb
class Item::Sham
  def self.options
    { :quantity => 10, :weight => 1.0, :price => 10.0, :name => Sham.string! }
  end
end

# in sham/line_item_sham.rb
class LineItem::Sham
  def self.options
    { :quantity => 1, :item => Sham::Base.new(Item) }
  end
end

# in sham/cart_sham.rb
class Cart::Sham
  def self.options
    { :user => Sham::Base.new(User) }
  end
end

# in sham/user_sham.rb
class user::Sham
  def self.options
    {
      :username => "#{Sham.string!}@example.com",
      :password => "password",
      :password_confirmation => "password"
    }
  end
end
{% endhighlight %}

## 4. Write Your Tests.

{% highlight ruby %}
describe Cart do
  it "should be able to calculate the total price" do
    cart = Cart.sham!
    li1 = LineItem.sham! :cart => cart
    li2 = LineItem.sham! :cart => cart
    cart.price.should == li1.price+li2.price
  end
end
{% endhighlight %}

Much Simpler!  Here are a couple things to keep in mind:

#### You can override attributes.

You can override any attributes that have been defined in a sham.  For example,
either of the following is perfectly acceptable:

{% highlight ruby %}
Item.sham! :quantity => 100
Item.sham!
{% endhighlight %}

#### You can create random strings.


`Sham.string!` will generates a random string that can be used as a filler or to
avoid validation issues.  For example if there is a requirement that your
usernames are unique, you can use Sham.string! to avoid users having the same
username.

Normally, this would fail because the two users would have the same username:

{% highlight ruby %}
User.create :username => "username@example.com"
User.create :username => "username@example.com"
{% endhighlight %}

But, this will not fail because each username will be globally unique:

{% highlight ruby %}
User.sham! :username => "#{Sham.string!}@example.com"
User.sham! :username => "#{Sham.string!}@example.com"
{% endhighlight %}

#### You can nest shams.


`Sham::Base.new(User)` tells Sham that when it's creating a model it should
create a User sham if one is not specified.  That means that either of these two
calls is valid:

{% highlight ruby %}
Cart.sham! :user => User.sham!(:username => "person@example.com")
Cart.sham!
{% endhighlight %}

Check out the [documentation](https://github.com/panthomakos/sham#readme) for
additional examples and features. Feel free to
[file an issue](https://github.com/panthomakos/sham/issues) if something is not
working as you would expect.  There's much more you can do with this gem and
while I do plan on continuing to develop features, they will remain lightweight
and simple.
