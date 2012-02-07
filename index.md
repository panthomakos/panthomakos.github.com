---
layout: default
title: a.blog.about.code
---

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>