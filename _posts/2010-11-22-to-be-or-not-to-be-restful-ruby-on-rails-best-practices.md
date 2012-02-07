---
layout: post
title: To be or not to be... RESTful - Ruby on Rails Best Practices
categories: [Best Practices]
tags: [controllers, rails, views]
---

Ruby on Rails assumes you will be developing your controllers and views with a
[RESTful architecture][rest] in mind.  What this means is that if you have a
table named `posts`, a model named `Post` and a controller named
`PostsController` you have some URLs automatically available to you.

[rest]: http://en.wikipedia.org/wiki/Representational_State_Transfer

  * HTTP GET /posts: List all the posts in the system.
  * HTTP GET /posts/[id]: Show me a specific post.
  * HTTP POST /posts: Create a new post.
  * HTTP PUT /posts/[id]: Update a specific post.
  * HTTP DELETE /posts/[id]: Remove a post.

In fact, creating the associated migration, model, controller and view is only
a couple of scaffold generation commands away.  Once you have this all setup
though, you begin to ask yourself "[What's so great about this?][so]"  Besides
being easy to create all of these files and hook them up, what's the actual
added benefit of structuring my code this way?  This only becomes more apparent
when you start adding additional functionality, for example you want an
`ArchivesController` that lists your blog entries by date.

[so]: http://stackoverflow.com/questions/4249635/rails-restful-resources-worth-using-or-inflexible-overrated

  * HTTP GET /archives: Show me the archived years.
  * HTTP GET /archives/[year]: Show me the archived months for a year.
  * HTTP GET /archives/[year]/[month]: Show me the posts for a specific month and year.

You realize that you don't want all the fancy PUT, POST, DELETE actions, and
your URL structure is actually entirely different from the RESTful architecture.
Do you still have to keep using REST? Should you restructure your
`ArchivesController` to fit the RESTful architecture?

The short answer is no, you don't need to use REST.  REST is great for some
projects and some entities, but not for others.  Generally it's also a good
starting point, and more importantly it creates a basic protocol that you or
other developers, in the case of an API, can depend on.  It doesn't require you
to remember what you need to do to create a new entity, you simply make a POST
request.  That said it's not the be-all-and-end-all of software architecture.
For example, a web-site's marketing pages (/about, /tour, /pricing,
/special-offer-december) usually do not fit well into a RESTful architecture,
and additional functionality like search (HTTP Get /posts/search) or stats (HTTP
Get /posts/stats) is often added in parallel to the RESTful architecture.

Before you get started, here are some questions you might want to ask yourself:

  * Does this controller/view deal primarily with an object/entity like a Post, Blog, User, Comment?
  * Are create, update, delete, edit and new actions all going to be available on the web?

As a guideline, if you answer YES to these two questions, then it's probably
best to start with REST and expect that you will eventually use the architecture
as a building block for additional actions and views you might want to perform.
Otherwise, pick a URL that best represents what the action will show or do
(/archives, /tour, /december-offer) and make sure you use the proper HTTP
Protocols (GET for display, PUT for update, DELETE for removing and POST for
creating).
