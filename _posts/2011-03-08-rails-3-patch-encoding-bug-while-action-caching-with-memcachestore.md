---
layout: post
title: 'Rails 3 Patch: Encoding Bug while Action Caching with MemCacheStore'
categories: [Review]
tags: [rails, ruby]
---

While upgrading an application from Rails 2.3 and Ruby 1.8.7 to Rails 3.0 and
Ruby 1.9.2 I ran into the following error message:

    Encoding::CompatibilityError: incompatible encoding regexp match
    (ASCII-8BIT regexp with UTF-8 string)

After some searching around I was able to determine that this was a
`MemCacheStore` issue. In particular it was caused when trying to perform action
caching on a URL that contained special UTF-8 encoded characters, like the
umlaut.  As luck would have it
[someone else was experiencing this same issue][issue], but their lighthouse
ticket had been around for a good two months without any activity. I guess not
too many Ruby on Rails developers are performing action caching on pages with
UTF-8 encoded parameters. Anyways, I took it upon myself to create a patch to
solve this issue.


The bug was located in ActiveSupport's `mem_cache_store.rb` on line 33, where
the following regular expression was defined:

{% highlight ruby %}
ESCAPE_KEY_CHARS = /[\x00-\x20%\x7F-\xFF]/
{% endhighlight %}

By itself this doesn't seem like much of an issue, but what you might not
realize is that the Rails source code is encoded in US-ASCII, so when your UTF-8
encoded string is gsubbed with this regular expression, you get an
`Encoding::CompatibilityError` on the first line of the
`ActiveSupport::Cache::MemCacheStrore#escape_key` function:


{% highlight ruby %}
def escape_key(key)
  key = key.to_s.gsub(ESCAPE_KEY_CHARS){|match| "%#{match.getbyte(0).to_s(16).upcase}"}
  key = "#{key[0, 213]}:md5:#{Digest::MD5.hexdigest(key)}" if key.size > 250
  key
end
{% endhighlight %}

My solution to this problem was to change the escape_key function so that it was
compatible with any encoded string.  I decided that the simplest solution would
be to force the memcached key into the same encoding as the regular expression,
but to do that I would need to duplicate the key string, because performing a
force_encoding on a string actually changes the string in memory, it doesn't
create a new string.  If the string is not duplicated, the original key will get
sent back to the Rails application with a different encoding and this can cause
issues if you perform any concatenation with that string, since most of the
strings in your app will probably be encoded in UTF-8. Try this out for yourself
in you console:

{% highlight irb %}
irb(main):001:0> foo = "bar"
=> "bar"
irb(main):001:0> foo.encoding
=> #<Encoding:UTF-8>
irb(main):001:0> foo2 = foo.force_encoding(Encoding::ASCII_8BIT)
=> "bar"
irb(main):001:0> foo.object_id == foo2.object_id
=> true
irb(main):001:0> foo.encoding
=> #<Encoding:ASCII-8BIT>
{% endhighlight %}

Once the string was encoded in ASCII-8BIT, it was compatible with the regular
expression and no error would be raised. Here's the final function I came up
with:

{% highlight ruby %}
def escape_key(key)
  # Fix for EncodedKeyCacheBehavior failing tests in caching_test.rb.
  key = key.to_s.dup
  if key.respond_to?(:encoding) && key.encoding != ESCAPE_KEY_CHARS.encoding
    key = key.force_encoding(ESCAPE_KEY_CHARS.encoding)
  end
  key = key.gsub(ESCAPE_KEY_CHARS){|match| "%#{match.getbyte(0).to_s(16).upcase}"}
  key = "#{key[0, 213]}:md5:#{Digest::MD5.hexdigest(key)}" if key.size > 250
  key
end
{% endhighlight %}

Please give my patch a try, it includes tests for all encoding types and special
characters. If you get your tests passing please leave a comment
[on the lighthouse ticket][issue] that you were able to verify the fix.

Update - [This patch](https://github.com/rails/rails/pull/219) has been merged
into the rails branch as of April 28, 2011.

[issue]: https://rails.lighthouseapp.com/projects/8994/tickets/6225-memcachestore-cant-deal-with-umlauts-and-special-characters
