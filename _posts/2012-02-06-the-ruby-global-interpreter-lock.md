---
layout : post
title : The Ruby Global Interpreter Lock
category : Under the Hood
tags : [gil, ruby, threads]
---
All of the code in this post was executed on Ruby 1.9.3 in February 2012.
Ruby MRI refers to the C implementation of Ruby.

The Ruby MRI Global Interpreter Lock (GIL) is often misunderstood. Programmers
hear "global" and "lock" and assume that threaded code won’t execute with any
concurrency. While I'm hopeful that it will eventually be removed (jRuby and
Rubinius Hydra) the GIL is not all that bad, and the state of Ruby MRI
threading can only improve.

The reason I say it's not that bad is that the GIL only applies to Ruby
operations. That means it really only applies to the work that Ruby is doing.
Which makes sense, because it exists to protect the integrity of the Ruby
Virtual Machine.

But besides Ruby operations, what kind of other work is there? In a typical web
application – a lot. When you're querying the database, disk or memcached Ruby
isn't doing work, which means you aren't invoking the GIL. Let's take a look at
a simple example.

## I/O Operations

{% highlight ruby %}
require 'benchmark'
require 'mysql2'

x = Mysql2::Client.new
y = Mysql2::Client.new

Benchmark.bm do |b|
  b.report('w/o') do
    x.query("SELECT SLEEP(1)")
    y.query("SELECT SLEEP(1)")
  end

  b.report('with') do
    a = Thread.new{ x.query("SELECT SLEEP(1)") }
    b = Thread.new{ y.query("SELECT SLEEP(1)") }
    a.join
    b.join
  end
end

       user     system      total        real
w/o  0.000000   0.000000   0.000000 (  2.002874)
with  0.000000   0.000000   0.000000 (  1.001658)
{% endhighlight %}

In this example Ruby passed the SQL query into a C program that waited on the
database to respond. During that time Ruby released the GIL because it was not
performing any work. Querying the database is an I/O operation and the GIL does
not factor in to I/O operations. That means that the Ruby interpreter had the
freedom to process more than one thread at a time. To understand the difference,
let’s take a look at this next example.

## Ruby Operations

{% highlight ruby %}
require 'benchmark'

Benchmark.bm do |x|
  x.report('w/o') do
    10_000_000.times{ 2+2 }
  end

  x.report('with') do
    a = Thread.new{ 5_000_000.times{ 2+2 } }
    b = Thread.new{ 5_000_000.times{ 2+2 } }
    a.join
    b.join
  end
end

       user     system      total        real
w/o  2.300000   0.000000   2.300000 (  2.312023)
with  2.330000   0.000000   2.330000 (  2.338784)
{% endhighlight %}

In this example Ruby had to perform computations and modify data, so threading
provided no benefit since the GIL was active. In fact attempting to parallelize
these computations incurred the overhead of creating two threads.

## Summary

The Ruby GIL only applies to non I/O operations. While removal of the GIL would
allow my last example to take full advantage of native threads, Ruby threading
is not in a state of non-existence. Ruby threading does work, it's just hasn't
evolved to the same level as other languages yet.

Hopefully this clarified the GIL and threading. At this point in time
event-driven ruby, threads and fibers provide a great deal of concurrency in
Ruby despite the GIL. I believe in the future this will only improve –
particularly for threads. The GitHub Gist of this code also includes another
example using Ruby’s sleep function.
