---
layout: post
title: Serving Static Files with Passenger and Nginx
category : Tutorial
tags : [nginx, passenger, rails]
---

The [X-Accel-Redirect HTTP header][x-accel] is used on Nginx servers to allow
applications to serve static files directly from Nginx.  For those familiar with
[X-Sendfile](http://john.guen.in/rdoc/x_send_file/), X-Accel-Redirect is the
Nginx implementation. Using this header to serve static files from a Rails
application has some great benefits, like not requiring a file to be streamed
through the [Passenger](http://www.modrails.com) and Rails stacks - which can be
costly.  The setup is quite simple.

[x-accel]: http://wiki.nginx.org/NginxXSendfile

<ol>
  <li>
Create a controller to serve the file.
{% highlight ruby %}
class BlogsController < ApplicaitonController
  def show
    @blog = Blog.find(params[:id])
    response.headers["X-Accel-Redirect"] = @blog.path
    response.headers["Content-Type"] = "application/json"
    render :nothing => true
  end
end
{% endhighlight %}
  </li>
  <li>
Write an nginx location block into nginx.conf.
        server {
          ...
          location /blogs/ {
            internal;
            root /data;
          }
        }
  </li>
  <li>
Serve the file.
{% highlight ruby %}
blog = Blog.create(:path => "/blogs/my_first_blog.json")
uri = URI.parse("http://localhost:3000/blogs/#{blog.id}.json")
Net::HTTP.get uri
{% endhighlight %}
  </li>
</ol>

The request goes something like this:

  * The HTTP GET request goes to Nginx, Passenger, and finally to your Rails
    BlogsConroller#show action.
  * The BlogsController#show action responds with two HTTP headers
    (X-Accel-Redirect and Content-Type).
  * The X-Accel-Redirect header is trapped on the return trip by Nginx.
  * Nginx matches the X-Accel-Redirect header with the location /blogs/ block.
  * Nginx serves the file directly from /data/blogs/my_first_blog.json.
  * Nginx also persists the [Content-Type HTTP header][content-type] and passes
    it to the client.

[content-type]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17

An additional benefit of this method is that you can use the controller to check
file permissions.  For example:

{% highlight ruby %}
class BlogsController < ApplicaitonController
  def show
    @blog = Blog.find(params[:id])
    if current_user.can_access?(@blog)
      response.headers["X-Accel-Redirect"] = @blog.path
      response.headers["Content-Type"] = "application/json"
      render :nothing => true
    else
      redirect_to permission_denied
    end
  end
end
{% endhighlight %}
