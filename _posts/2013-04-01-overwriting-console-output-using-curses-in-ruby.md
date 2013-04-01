---
layout: post
title: Overwriting Console Output Using Curses in Ruby
category: Tutorial
tags: [ruby]
---

[Καλό μήνα][greeting]! A while back I wrote about how to
[overwrite console output in Ruby][article]. Originally the project I used this
technique for only required overwriting a single line of output. That project
has now grown and I need to be able to maintain a mini-dashboard on the console.

[greeting]: http://translate.google.com/?hl=en&tab=TT&authuser=0#el/en/%CE%9A%CE%B1%CE%BB%CF%8C%20%CE%BC%CE%AE%CE%BD%CE%B1
[article]: /2012/04/16/overwriting-console-output-in-ruby

Fortunately, Ruby rocks and has a [standard library wrapper for Curses][curses].
I would encourage you to read through the documentation. I've written a simple
script to show-off the most basic use-case. In this example I initialize ten
workers. Each worker is responsible for keeping track of the work she has
completed, `@percent`, and then reporting that progress on her own line,
`@index`, of the console.

[curses]: http://www.ruby-doc.org/stdlib-2.0/libdoc/curses/rdoc/Curses.html

{% highlight ruby %}
#!/usr/bin/env ruby

require 'curses'

Curses.noecho
Curses.init_screen

class Worker
  def initialize(index)
    @index = index
    @percent = 0
  end

  def run
    (1..10).each do
      work
      report
      sleep(rand())
    end
  end

  def to_s
    "Worker ##{'%2d' % @index} is #{'%3d' % @percent}% complete"
  end

  private

  def work
    @percent += 10
  end

  def report
    Curses.setpos(@index, 0)
    Curses.addstr(to_s)
    Curses.refresh
  end
end

workers = (1..10).map{ |index| Worker.new(index) }

at_exit do
  workers.each{ |worker| puts worker }
end

workers.map{ |worker| Thread.new{ worker.run } }.each(&:join)

Curses.close_screen
{% endhighlight %}

Because Curses clears the screen once it is complete, the `at_exit` block
ensures that the last known state of the workers is echoed to the screen one
last time.

Another command I found useful was the `Curses.clear` command. This function
clears the entire screen before rewriting output. This is particularly helpful
when you want to overwrite a long line of text with a shorter one.

I hope this was helpful! Have fun making some cool console applications. If you
build something worth sharing, please leave a comment that links to your code.
