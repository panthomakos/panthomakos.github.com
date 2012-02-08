---
layout: post
title: Hoptoad Error Tracking in Ruby on Rails
category: Review
tags: [hoptoad, rails]
---

What do you do with error messages from your production servers? I used to have
them all sent to my inbox.  I would then have to filter them based on if they
were coming from delayed jobs or what part of the site they were relevant to. I
would then have to sift through this massive email to find the stack-trace of
the environment variables, it was just a pain in the ass.  I recently started
using [Hoptoad](http://hoptoadapp.com/pages/home) to track my error messages.
Life has been much easier since that decision. Hoptoad is an online application
that acts as an API for receiving your error messages, as well as website for
tracking, grouping and resolving those error messages.

The Hoptoad interface is quite simple: related errors are grouped together and
you can 'resolve' them with a simple switch to indicate that you have taken care
of the issue.  This is awesome because you don't have to resolve 10 error
messages when there's really only one thing that's going wrong. Hoptoad also
tracks deployments and errors/minute for some additional fun stats. For
organization it splits your error summary, stack trace, environment variables,
request parameters, session variables and similar errors into separate tabs for
easy navigation. Hoptoad is a [paid service](https://hoptoadapp.com/account/new)
but they offer a free plan for a single project, single user and five error
messages per minute so that you can try the service out, and their 'tadpole', or
smallest paid, plan comes in at $5 per month.

My favorite gem to get started using Hoptoad is
[hoptoad_notifier](https://rubygems.org/gems/hoptoad_notifier). You can get
detailed installation instructions
[here](https://github.com/thoughtbot/hoptoad_notifier). My setup goes something
like this:

<ol>
<li>
Add the gem to my Gemfile.
{% highlight ruby %}gem 'hoptoad_notifier'{% endhighlight %}
</li>
<li>
Install my bundle.
{% highlight bash %}bundle install{% endhighlight %}
</li>
<li>
Generate my api key yaml file.
{% highlight bash %}rails generate hoptoad --api-key{% endhighlight %}
</li>
<li>
Add a deploy notification to my after_restart script.
{% highlight ruby %}
run "cd #{release_path} && rake hoptoad:deploy
TO=#{@configuration[:environment]}
REVISION=#{@configuration[:revision]}
REPO=#{@configuration[:repository]}"
{% endhighlight %}
</li>
<li>
Add a notification handler to my delayed jobs.
{% highlight ruby %}
# in config/initializers/delayed_job_with_hoptoad.rb
class Delayed::Worker
  alias :handle_failed_job_without_hoptoad :handle_failed_job

  def handle_failed_job(job, error)
    HoptoadNotifier.notify(error)
    handle_failed_job_without_hoptoad(job, error)
  end
end
{% endhighlight %}
</li>
</ol>

You're set to go. All errors in your app and in your delayed job processes will
get posted to Hoptoad, and Hoptoad will track your deployments.
