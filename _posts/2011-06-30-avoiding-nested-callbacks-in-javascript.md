---
layout: post
title: Avoiding Nested Callbacks in JavaScript
category: Best Practices
tags: [javascript, node.js]
---

I was reading through the
[Mastering Node E-Book](http://visionmedia.github.com/masteringnode), which is a
great resource for getting started with node.js by the way, and I came across
this code example.

{% highlight javascript %}
var fs = require('fs');

fs.mkdir('./hello',0777,function(err){
  if (err) throw err;

  fs.writeFile('./hello/world.txt', 'Hello!', function(err){
    if (err) throw err;
    console.log('File created with contents: ');

    fs.readFile('./hello/world.txt', 'UTF-8', function(err, data){
      if (err) throw err;
      console.log(data);
    });
  });
});
{% endhighlight %}

This code creates a directory `./hello`, then writes a file `./hello/world.txt`,
and then reads the contents of that file. All the filesystem operations are
handled through callbacks - that's the way we do things in JavaScript. The
example is quite simple and while it's specific to node, it illustrates a pretty
common case of deeply nested callbacks in JavaScript. This set of instructions
might be easy enough to read here, but it doesn't take a genius to figure out
that these nested callbacks will eventually become a bit of a headache.

Fortunately there is a better way - named callbacks. Here is how the latter code
can be transformed into a less-nested program.

{% highlight javascript %}
var fs = require('fs');

fs.mkdir('./hello',0777,makeDirectory);

function makeDirectory(err){
  if (err) throw err;
  fs.writeFile('./hello/world.txt', 'Hello!', writeFile);
}

function writeFile(err){
  if (err) throw err;
  console.log('File created with contents: ');
  fs.readFile('./hello/world.txt', 'UTF-8', readFile);
}

function readFile(err, data){
  if (err) throw err;
  console.log(data);
}
{% endhighlight %}

Pretty cool, huh? Well, until you have two or three callbacks that respond
differently to fs.readFile and you start running out of function names, right?
Not to worry, you can create closures for your callbacks as well. For example,
let's say we want to return the result of fs.writeFile, but we want to write two
different files, namely `./hello.txt` and `./world.txt`. While this example is a
little inane it demonstrates how easily the closures help to partition your code
without having to rely solely on unnamed callbacks.

{% highlight javascript %}
var fs = require('fs');

function writeHello(){
  return fs.writeFile('./hello.txt', 'Hello!', writeFile);

  function writeFile(err){
    if (err) throw err;
    console.log('Wrote hello.txt');
  }
}

function writeWorld(){
  return fs.writeFile('./world.txt', 'World!', writeFile);

  function writeFile(err){
    if (err) throw err;
    console.log('Wrote world.txt');
  }
}

writeHello();
writeWorld();
{% endhighlight %}

Wait, how did that work, didn't we return before defining that function? Yes,
but not quite, in JavaScript any functions defined using `function name(){}` as
opposed to using `var` are available immediately upon entering the function or
program body respectively. That means that we can define named callbacks after
our return statements and still have a linear, readable and less-nested program.
You can find a [gist of this code on github](https://gist.github.com/1055795),
enjoy!
