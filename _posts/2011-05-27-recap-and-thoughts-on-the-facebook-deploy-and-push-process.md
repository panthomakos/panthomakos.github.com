---
layout: post
title: Recap and Thoughts on the Facebook Deploy and Push Process
categories: [Best Practices]
tags: [tdd, facebook, git]
---

Yesterday, Facebook gave a presentation on their front-end push/deploy process
called [Pushing Millions of Lines of Code Five Days a Week][facebook]. Facebook
releases code once a week, but they push new code daily. While few companies
operate on this scale there is a lot we can learn from their "push culture" and
the tools they use to promote a bug free deploy process.

[facebook]: https://www.facebook.com/event.php?eid=209798352393506

## Culture

There was a large focus on push culture. While tools are great and can help,
they don't mean a thing if there isn't a company culture to support a bug-free
and painless deploy process.  Creating this culture early on is important, that
includes ensuring that developers are on the hook for the changes they make.
The further the developer is from the end product and release date, the less
likely they are to be held accountable for their code.  Pushers at Facebook
ensure that a developer is responsible for their code that day, that week and
even if they move to another group within Facebook.

## Tools

For the geekier viewer the tools part is always fun - we love gadgets! Facebook
has built a suite of monitoring tools they use internally to ease the push
process.  For the most part there are free, open-source and DIY alternatives to
a lot of these tools.  Facebook is at a much larger scale than many of us, so in
my Take Away section I am going to focus more on what you can do to implement
these tools and the culture they promote.

* **IRC Bots** are used because all internal communication is done through IRC,
so it's important to provide automated answers to some of the most commonly
asked questions and avoid overwhelming the push engineers. For instance, if a
developer wants to know, "will my revision be in this push?" they need only ask
an IRC bot.
* **Automated Tests** are critical because engineers are responsible for writing
unit and selenium tests to support their code. These tests are automated and
they are used to determine if a revision is ready for release.
* **Shadow Branches** are pre-live versions of Facebook. Internally facebook.com
points to latest.facebook.com - a version of facebook that is ready for release
this week. These shadow branches ensure the "you are always testing" mentality
and help ferret out bugs before the push begins and during the first phases of
the deploy process.
* **Error Tracking** is controlled via a data-centric internal dashboard that
includes information on where the error happened and which developer is to blame
for that line of code.
* **GateKeeper** is a fancy tool that developers use to roll-out features to
subsets of users.  Subsets can include percentages of the population, males or
females only, people with or without a certain affiliation or group membership
and more.
* **Perflab** is a tool for tracking performance changes on code branches and
revisions and the impact your change has on the site.
* **HipHop** is a compiled version of PHP that Facebook has built internally.
It decreases server load by about 50% and turns facebook.com into a 1GB binary
file rather than a collection of PHP files. Facebook compiles in about 10
minutes.
* **BitTorrent** has been modified to distribute the HipHop binary to all
clusters and within clusters of machines.  It is used to roll out changes
quickly and efficiently. A new Facebook build can be rolled out in about 15
minutes on all machines.
* **Push Karma** is a tool that is used to privately track how reliable an
individual developer is.  It was emphasized that this is not a public shaming
tool, it is simply used to determine the chance that an individual developer's
changes might mess things up.  If someone is more prone to bugs then a last
minute change is less likely to be accepted into the release.
* **SVN/Git** are the SCMs of choice. The central trunk repository is SVN,
developers use Git for daily work and feature development.

## Take Away

The main take away is that if you want to focus on a bug-free push culture it's
important to get developers on the hook for their changes and bring them closer
to the final product.  The more QA and administration hurdles a piece of code
has to go over, the less likely this is to happen.  If a developer can see and
use his or her changes they are more likely to feel responsible for them and
they are more likely to catch bugs. The most important tools that promote this
mentality are the shadow branch, error tracking and push karma. The other three
tools that I believe are of great importance to a web-application company are
TDD/BDD, GateKeeper and Performance Monitoring.

#### The Shadow-Branch

The easiest way to implement shadow-branch is to have a staging server.  If you
don't already do this, you should. Staging servers are a great way to ensure
that the code you are releasing works in an environment that mimics production
as closely as possible. This usually also means using a live or replicated
version of a live dataset, an external url (even if it's only internally
available), and replicating things like content-delivery networks and user
access patterns.

#### Error Tracking

An error tracking tool is also critical, and it's important to dedicate someone
on your team to track these errors.  If you can't automate notifications so that
individual developers are notified of a bug, it's important to designate one
person the task of monitoring errors on your site.  If you don't know there are
errors, you aren't going to fix bugs.

#### Push Karma

Push Karma is not as critical and I wouldn't even recommend building a tool to
automate this process, but it's important that someone is aware of who is
introducing bugs and who is responsible for them.  Some adult person needs to be
responsible for determining if a developer has a higher chance of creating bugs,
and if they have gone through the appropriate code-review processes and test
driven development processes.  This ensures that last minute changes and large
releases are smoother. It's important to not publicly or privately shame people.
I think creating a stressful environment around bugs is not healthy. You want
developers that are creative, happy and not stressed about introducing bugs. But
it is important to know who needs a little extra code review and a little bit
more time to release a feature.

#### Test/Behavior Driven Development

I think the test automation speaks for itself.  If you are developing a large
web application you cannot manually test everything and you cannot determine all
of the side-effects of your code.  Test/Behavior Driven Development means you
are ensuring your code will work now and later down the road when someone else
makes a change. It's just common sense.

#### GateKeeper

I believe it's good to build features with an off-switch, if something goes
wrong you want to be able to turn things off.  Especially with the prevalence of
companies hosting their applications in the cloud, it's important to think about
how you can keep your site running when code or an external service doesn't
work.  Every feature should have an off switch that doesn't require bringing
down your website and re-booting.

#### Performance Monitoring

Studies show that small (under one second) degrade in page performance can
result in users walking away.  If a user is on your website for entertainment,
in other words for something other than checking their bank account, you need to
ensure that your site is performant. Make sure you either test performance or
monitor it with tools like NewRelic.
