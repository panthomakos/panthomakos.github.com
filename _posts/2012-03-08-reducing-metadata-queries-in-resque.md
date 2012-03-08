---
layout: post
title: Reducing Metadata Queries in Resque
category: Tutorial
tags: [rails]
---

If you're using Resque with your Rails application you may have noticed a large
number of `SHOW TABLES`, `SHOW CREATE TABLE` and `SHOW FIELDS FROM` queries in
your log while a job is processing. The most frustrating thing about this is
that they probably don't appear in your web requests.

The solution for this is quite simple. Because Resque forks its processes before
each job is executed it only maintains - in memory - the cached metadata that
was loaded before the fork. To resolve this issue we need to load all this
metadata before the Resque job forks. Fortunately Resque provides a
`before_first_fork` hook which allows us to execute code before the first job,
hence allowing us to pre-load a lot of metadata into memory.

Add the following to an initializer file, like `config/initializers/resque.rb`.

{% highlight ruby %}
# Pre-load all the ActiveRecord column metadata.
Resque.before_first_fork do
  ActiveRecord::Base.send(:subclasses).each do |klass|
    unless klass.abstract_class?
      klass.primary_key
      klass.columns
    end
  end
end
{% endhighlight %}

This code loads all the `ActiveRecord` subclasses and then, if they have a table
associated with them, loads their column and primary key information. The column
information is associated with the `SHOW FIELDS FROM` queries and the primary
key information is associated with the `SHOW TABLES` query.

A note of caution. If you happen to use a gem that also hooks into the Resque
pre-fork process you may run into issues. Currently Resque only supports a
single block/code snippet per hook. Hopefully this issue will be resolved in
[this pull request][pull] that I created yesterday.

[pull]: https://github.com/defunkt/resque/pull/537

But if you are itching to get this working right away you can use my branch that
contains the updates by adding the following to your Gemfile:

{% highlight ruby %}
gem 'resque', git: 'git://github.com/panthomakos/resque', branch: 'hooks'
{% endhighlight %}

This code allows Resque to keep track of, and register, multiple fork hooks, so
gems like [rpm contrib][gem], which also hook into the Resque boot process, can
co-exist with your hooks.

[gem]: https://github.com/newrelic/rpm_contrib

Enjoy the faster Resque processing!
