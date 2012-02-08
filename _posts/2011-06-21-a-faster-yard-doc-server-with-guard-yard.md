---
layout: post
title: A Faster YARD Doc Server with Guard-Yard
categories: [Tutorial]
tags: [guard, ruby, rails, yard]
---

I recently started using [YARD](http://yardoc.org/) to view my documented source
code in an HTML format.  I really like the
[syntax and navigation](http://rubydoc.info/gems/timezone/0.1.2/frames) that the
web interface provides, and I feel that having this nicely formatted display of
documentation really pushes me to think more about commenting my code while
developing. Starting the YARD server locally is as easy as installing the gem,
generating the docs and starting the server.

But I found that it's a bit of a pain in the ass to actually view documentation
while it's undergoing changes. While you can run the server in "reload" mode,
this requires your entire documentation to be re-generated on every request, and
I feel that it's wildly inefficient and slow. To remedy this problem I released
a gem that integrates with [guard](https://github.com/guard/guard) to bring you
a speedy and up-to-date YARD Documentation Server.

There are two major benefits to the
[guard-yard](https://github.com/panthomakos/guard-yard) gem.  The first is that
it provides a consolidation of all file-monitoring processes to guard. This
means you only need to run guard to keep your documentation server up-to-date
and running. There is no need to configure the YARD server or re-processes your
documentation on every request. The second benefit is that the docs are updated
quickly. The guard-yard gem only modifies the documentation for files that have
actually changed. This means that viewing your changes happens sooner!

Staring the YARD server via guard-yard is very easy:

1. Install [guard](https://github.com/guard/guard),
[guard-yard](https://github.com/panthomakos/guard-yard), and
[yard](https://github.com/lsegal/yard) gems.
2. Update your Guardfile using `guard init yard`.
3. Start the Guard server using `guard start`.

For additional details check out the
[gem page](https://rubygems.org/gems/guard-yard).
