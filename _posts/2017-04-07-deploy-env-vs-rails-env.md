---
layout: post
title: Deploy Env Vs Rails Env
---

Reading through [Config: Behavior versus Credentials](https://www.schneems.com/2017/03/21/config-behavior-versus-credentials/) I was struck that, at Strava, we ran into this very same problem in our early Rails days. It didn't take us long to realize:

> `RAILS_ENV` is meant to configure the behavior of the Rails framework - not your application.

The clearest example of this distinction, as [Richard Schneeman](https://twitter.com/schneems) points out, is credentials. You do not want to tie credentials to a Rails environment. Credentials, regardless of how you manage them, should be configured based on the deployment environment.

On the other side of this spectrum is `ActiveSupport` autoloading. Autoloading is a framework responsibility. You should tie autoloading behavior to the Rails framework environment - not to your deployment environment.

In practice, what does this mean? You effectively should specify two environment variables when running your application:

        RAILS_ENV=production DEPLOY_ENV=production ./run

        RAILS_ENV=production DEPLOY_ENV=staging ./run

In reality the `DEPLOY_ENV` might be a set of environment variables or a pointer to a configuration store. The core concept is that `DEPLOY_ENV` is a different dimension of configuration than `RAILS_ENV`.
