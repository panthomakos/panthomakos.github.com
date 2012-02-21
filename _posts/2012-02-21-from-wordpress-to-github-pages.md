---
layout: post
title: From WordPress to GitHub Pages
category: Review
---

A little while ago I migrated ablogaboutcode.com from a [wordpress.com][wp]
hosted site to a [jekyll][jekyll]-generated and [github-pages][pages]-hosted
site. This is a move I've been wanting to make for quite some time now. My
reasons were primarily due to control over content and style combined with
wanting to own the code behind it all.

[wp]: http://www.wordpress.com
[jekyll]: https://github.com/mojombo/jekyll
[pages]: http://pages.github.com/

Wodpress served me well when I was first getting started - and trying to decide
if I wanted to commit to blogging and producing content on a regular basis. It
was also a great way to familiarize myself with a full-featured blogging
platform and what that could potentially provide. But wordpress.com imposes some
constraints around commenting and formatting that I wasn't going to be happy
with in the longer term.

I could have used my own wordpress installation, but I think that as a blogging
platform it's overly complex and slow. And I like coding. Moving to jekyll means
that I can write my blog posts in vim, using markdown, and then easily check the
results on a local server - no internet required. I'm actually writing this
particular blog post on a bus with non-functioning internet.

The migration process was relatively painless. Disqus provides a great import
tool for migrating comments from wordpress. Transferring my domain registration
from wordpress.com to GoDaddy was also a simple process.

The hardest part of the move was converting my posts into markdown. There are
lots of tools to do this but in the end I still had to go through and clean up
each post individually. This consisted primarily of converting code and
highlight blocks to pygments and making sure that the formatting of my content
was to my liking - 80 character line lengths and pretty. Vim macros really made
this part of the process fast.

I started with the [Tom Preston][css] css template and added some of my own
customizations for Disqus and a more fluid layout. I've also added a floating
share bar which I plan on expanding in the future.

[css]: http://tom.preston-werner.com/

Enjoy!
