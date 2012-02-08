---
layout: post
title: How Javascript Loading Works - DOMContentLoaded and OnLoad
categories: [Best Practies, Tutorial]
tags: [dom, javascript, jQuery, performance]
---

There are actually three different windows of time during a web request in which
we can load and execute javascript. These windows are delineated by the
DOMContentLoaded event and the OnLoad event. We can load our scripts before
DOMContentLoaded, after DOMContentLoad, and after OnLoad. But before I dig into
the details of how this is done, what the heck are these two events and when are
they actually triggered?

## The Events

The [DOMContentLoaded][dom-content-loaded] event is triggered when the page's
Document Object Model (DOM) is ready.  What this means is that the API for
interacting with the content, style and structure of a page is ready to receive
requests from your code. In general this means that the text, css and html are
ready - but it's not quite that simple. Depending on the browser, HTML version,
and where the stylesheet tags are relative to the script tags, the CSS might not
be done rendering by the time this event fires. Fortunately there is a safe and
performant way to ensure that the CSS is loaded first. It's as simple as placing
your external stylesheets in the header and your external javascripts in the
footer. This approach forces the javascript to execute only after the external
stylesheets and HTML body have been loaded. Basically, this means that if you
are placing stylesheets in the header and javascripts in the footer, your DOM
will load all content, structure, and style before the DOMContentLoaded event is
fired. That's a good thing!

[dom-content-loaded]: https://developer.mozilla.org/en/Gecko-Specific_DOM_Events#DOMContentLoaded

The [OnLoad][on-load] event is triggered when the entire page has loaded. This
is moment when the globe or loader in your browser's title bar stops spinning.
At this point all scripts and stylesheets have finished loading and have been
executed, and all images have been downloaded and displayed.

[on-load]: https://developer.mozilla.org/en/XUL_Tutorial/More_Event_Handlers#Load_Events

## Loading Javascript

So how do we hook into each of these events; and more importantly, when should
we use each? To examine the different hooks, let's have a look at this simple
HTML page and the output that it produces.

{% highlight html %}
<!DOCTYPE html>
<body>
  <script src="jquery.min.js"></script>
  <script src="external.js"></script>
  <script type='text/javascript'>
    //<![CDATA[
      $.getScript('ajax.js');
      $(document).ready(function(){
        console.log('DOM Loaded (DOMContentLoaded)');
        $('head').append('<script src="dom.js"></script>');
        $.getScript('dom_ajax.js');
      });
      $(window).load(function(){
        console.log('Page Loaded (OnLoad)');
        $.getScript('page_ajax.js');
      });
    //]]>
  </script>
</body>
{% endhighlight %}

If we examine the javascript console we can see the list of events play out like
this:

    external.js Loaded and Executed
    DOM Loaded (DOMContentLoaded)
    dom.js Loaded and Executed
    Page Loaded (OnLoad)
    ajax.js Loaded and Executed
    dom_ajax.js Loaded and Executed
    page_ajax.js Loaded and Executed

This console output implicates the three windows of javascript loading:

* External javascript (external.js) is loaded before DOMContentLoaded.
* External javascript inserted into HTML during the DOMContentLoaded callback
(dom.js) is loaded before OnLoad.
* AJAX requests do not delay DOMContentLoaded or OnLoad, regardless of when they
are initiated (ajax.js, dom_ajax.js and page_ajax.js).

Let's take a look at each of these windows and see which one best suits our
needs.

## Before DOMContentLoaded

First, notice that both inline javascript and external javascript (external.js)
is loaded and executed immediately, regardless of the DOM being ready or the
page having been loaded.  This may seem obvious, but it is necessary to execute
this code before DOMContentLoaded so that we can actually hook into the event.
The best way to think of this javascript is as a setup. You shouldn't be
manipulating the DOM yet, since it isn't ready - you should be setting up your
callbacks and then getting out of the way so the rest of the page can load
quickly.

## After DOMContentLoaded

These scritpts (dom.js) are executed once the DOM is ready to be manipulated.
This where you want to put any code that actually changes the page visually as
this is the earliest point at which DOM manipulation is possible, and it will be
executed before the page is displayed. This is a good thing because we don't
want the page elements jumping around and changing style after they are visible
to the user.

## After OnLoad

These scripts (ajax requests and anything in the window.load callback) are
executed after the page has completed loading and the user can view the entire
document. This is a great place to load long-running scripts and scripts that
don't visually alter the page.  Some examples of this include analytics and
reporting libraries and code or data that might be used later and can be
pre-fetched for the user.  Google maps loads all of its map tiles using AJAX so
that the page load is not halted.

## Takeaway

Try to keep this page load process in mind when writing your javascript code.
Simple placement of your code can lead to large gains in overall perceived page
load time.  I'm hoping this blog post clarifies the basic page load events and
allows you to beter determine how and where javascript should be loaded in your
own application. If you have any questions or examples of these methods in use,
please leave a comment.
