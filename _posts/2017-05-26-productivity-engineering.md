---
layout: post
title: Productivity Engineering
---

On June 24th I'm giving a talk titled Developer Productivity Engineering (DPE) at [GORUCO](http://goruco.com/#speakers). DPE is a framework that applies the principles of [Site Reliability Engineering](https://en.wikipedia.org/wiki/Site_reliability_engineer), developed at Google, to improving productivity through automation.

While putting together my intro for this talk I realized that there are a few terms in the orbit of "productivity engineering" and it can often be confusing to know what's what.

You may have seen any one of these terms in a job description recently:

* DevTools Engineer
* Platform Engineer
* Release Engineer

Companies all tend to define these roles a little differently. I'm not going to try to clearly demarcate each title, but I think I've got a pretty good working definition and some distinctions that might help.

**Platform engineers** build systems on which other engineers build applications. Some canonical project examples might include implementing an event or queueing framework; or a scaling and maintaining a container orchestration cluster.

**DevTools engineers** build tools that help other engineers do their work. Some canonical project examples might include CLI scripts or IDE extensions; or a continuous integration and build service.

**Release engineers** build systems and create processes that enable engineers to ship code. Some canonical project examples might include automating a mobile release pipeline; or building a service to enable blue-green deployments.

Some organizations do not distinguish between these roles, they will often have a single Platform team. Other organizations are so large that they will have multiple siloed teams for each of these roles.

So where does productivity engineering fit in?

**Productivity engineers** build systems, create processes, and facilitate development efforts that enable other engineers to be more productive.

In a smaller company a productivity engineering team might own all of the example projects I listed above. In a larger organization the owned projects list may be smaller, but the team's contributions as facilitators and contributors to other teams will likely be much broader.

Productivity engineering is less focused on a single part of the development pipeline, and is instead focused on the bottlenecks in an organization. If the most pressing productivity slow-downs happen to be in a particular platform service that is not scaling, then productivity engineers have a role to play there.

At least thus far, this is how I see productivity engineering fitting into an engineering organization. The DPE framework is really a way of identifying, measuring, and prioritizing the work that will make everyone more productive.
