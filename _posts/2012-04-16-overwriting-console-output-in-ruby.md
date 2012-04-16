---
layout: post
title: Overwriting Console Output in Ruby
category: Tutorial
tags: [ruby]
---

I've always liked the way some long running console utilities display their
status without having to print a new line. For example the little spinner that
alternates displaying `|`, `/`, and `\`. Or various internet utilities that
display the percentage of the file that has been downloaded.

I recently wrote a utility to kick off parallel RSpec tasks on remote instances
and wanted to report the results back to the console after pinging the utility
every few seconds. I accomplished this with a cool little trick - the `\r`.

{% highlight ruby %}
(1..10).each do |percent|
  print "\r#{percent*10}% complete"
  sleep(0.5)
end
{% endhighlight %}

This little scripts begins by printing `10% complete` and works up to
`100% complete` in 10 percent increments. It's a cool way to notify the user
of your Ruby utility that the script is still progressing and that it hasn't
simply stopped dead in its tracks.

One thing to keep in mind is that you are overwriting the previous line, so
if your previous line is of a longer length than the next one, you might need to
pad it with whitespace to cover up the last message.
