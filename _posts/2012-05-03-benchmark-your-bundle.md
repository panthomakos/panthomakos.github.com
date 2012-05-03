---
layout : post
title : Benchmark Your Bundle
category : Under the Hood
tags : [ruby, rails]
---

Your Rails app takes anywhere from 20 seconds to a minute to boot.
[Maybe you've even tried to optimize your gem loading][post]. But which
gems are actually causing you all that trouble? How can you pinpoint the gems to
delay, replace or optimize?

[post]: /2012/01/12/a-simple-rails-boot-time-improvement/

Here's a simple script I wrote to benchmark the loading of each of the gems in
your Gemfile. A gist can be found [here][gist].

[gist]: https://gist.github.com/2588879

{% highlight ruby %}
#!/usr/bin/env ruby

require 'benchmark'

REGEXPS = [
  /^no such file to load -- (.+)$/i,
  /^Missing \w+ (?:file\s*)?([^\s]+.rb)$/i,
  /^Missing API definition file in (.+)$/i,
  /^cannot load such file -- (.+)$/i,
]

def pull(dep)
  begin
    Array(dep.autorequire || dep.name).each do |file|
      required_file = file
      Kernel.require file
    end
  rescue LoadError => e
    if dep.autorequire.nil? && dep.name.include?('-')
      begin
        namespaced_file = dep.name.gsub('-', '/')
        Kernel.require namespaced_file
      rescue LoadError
        REGEXPS.find { |r| r =~ e.message }
        raise if dep.autorequire || $1.gsub('-', '/') != namespaced_file
      end
    else
      REGEXPS.find { |r| r =~ e.message }
      raise if dep.autorequire || $1 != required_file
    end
  end
end

require 'rails/all'

$VERBOSE = nil

Benchmark.bm do |x|
  Bundler.setup.dependencies.each do |dependency|
    x.report(dependency.name[0..20].ljust(21)) do
      pull(dependency)
    end
  end
end
{% endhighlight %}

All of the code in `#pull` has been plucked right out of the
[Bundler require function][function]. It's responsible for loading
a gem depending on its Gemfile configuration. It considers custom require paths,
and generally converts dashes into slashes if it can't load the file.

[function]: https://github.com/carlhuda/bundler/blob/d351e68fa0a5df6de7aff587effce0801297848a/lib/bundler/runtime.rb#L51

The script then loads rails (`require 'rails/all'`) - something you won't be
able to avoid either way. It also turns warnings off (`$VERBOSE = nil`) so that
the benchmarking output looks a little nicer.

Once that setup is complete, it loops through each of the gem dependencies and
requires them. Your output will probably look something like this:

{% highlight ruby %}
rails                  0.000000   0.000000   0.000000 (  0.000215)
aws-s3                 0.160000   0.030000   0.190000 (  0.182991)
declarative_authoriza  0.850000   0.120000   0.970000 (  0.970152)
fog                    0.530000   0.060000   0.590000 (  0.589278)
mysql2                 0.010000   0.000000   0.010000 (  0.012908)
nokogiri               0.000000   0.000000   0.000000 (  0.000285)
oauth2                 0.140000   0.010000   0.150000 (  0.156330)
rack                   0.000000   0.000000   0.000000 (  0.000232)
rails_autolink         0.000000   0.000000   0.000000 (  0.000854)
rpm_contrib            0.450000   0.030000   0.480000 (  0.482697)
timezone               0.010000   0.000000   0.010000 (  0.007698)
jquery-rails           0.010000   0.000000   0.010000 (  0.008963)
hike                   0.000000   0.000000   0.000000 (  0.001602)
sass-rails             0.370000   0.020000   0.390000 (  0.400510)
coffee-rails           0.120000   0.020000   0.140000 (  0.136651)
uglifier               0.000000   0.000000   0.000000 (  0.002760)
capybara               0.010000   0.010000   0.020000 (  0.006783)
capybara-webkit        0.050000   0.000000   0.050000 (  0.053145)
database_cleaner       0.000000   0.000000   0.000000 (  0.007077)
gherkin                0.070000   0.010000   0.080000 (  0.068361)
jasmine                0.020000   0.000000   0.020000 (  0.025752)
jasmine-headless-webk  0.000000   0.000000   0.000000 (  0.010595)
launchy                0.060000   0.010000   0.070000 (  0.060992)
rspec                  0.350000   0.030000   0.380000 (  0.385097)
rspec-rails            0.000000   0.000000   0.000000 (  0.000903)
sham                   0.030000   0.000000   0.030000 (  0.025905)
spork                  0.000000   0.000000   0.000000 (  0.003553)
timecop                0.040000   0.000000   0.040000 (  0.041003)
guard                  0.000000   0.000000   0.000000 (  0.003354)
rb-fsevent             0.010000   0.000000   0.010000 (  0.004126)
guard-spork            0.000000   0.000000   0.000000 (  0.008679)
test-unit              0.130000   0.010000   0.140000 (  0.132788)
webmock                0.150000   0.020000   0.170000 (  0.167813)
vcr                    0.060000   0.000000   0.060000 (  0.068362)
{% endhighlight %}

I've condensed the output, but the total load time was 5 seconds. You can verify
that by benchmarking `Bundler.require` in your own application. You'll also
notice that the declarative authorization gem took up approximately 1 second
(20% of the total time) to load.  Maybe it's a good candidate for an upgrade,
replacement or optimization.
