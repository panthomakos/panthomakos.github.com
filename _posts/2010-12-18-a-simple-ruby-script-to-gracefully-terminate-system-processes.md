---
layout: post
title: A Simple Ruby Script to Gracefully Terminate System Processes
categories: [Tutorial]
tags: [ruby]
---

To gracefully terminate a system processes you issue the `kill -15` command. You
then usually monitor the process to ensure that it terminates.  If it does not,
you issue the `kill -9` command to ensure that it does.  Since this is a pretty
common task, I've created a simple Ruby script to automate it.

{% highlight ruby %}
#!/usr/bin/env ruby

require "timeout"

ARGV.each do |k|
    begin
        Process.kill("TERM", k.to_i)
        Timeout::timeout(30) do
            begin
              sleep 1
            end while !!(`ps -p #{k}`.match k)
        end
    rescue Timeout::Error
        Process.kill("KILL", k.to_i)
    end
end
{% endhighlight %}

This script will issue the `kill -15` command, monitor the process for up to 30
seconds to make sure it terminates, and if it still exists, issue the `kill -9`
command.

{% highlight bash %}gracefully-kill 1234{% endhighlight %}

You can also call this script for multiple process ids:

{% highlight bash %}gracefully-kill 1234 5678 91011{% endhighlight %}
