---
layout: post
title: Actually Always Refactoring
category: Refactoring
tags: [ruby, refactoring]
---

I apologize, it's been a while since my last blog post. After being sick, doing
a lot of climbing, and some traveling, I attended [GoGaRuCo 2012][gogaruco]. It
was my first Ruby conference and it really opened my eyes to the active and
friendly community of Ruby. It gave me the boost I needed to sit down and write
another blog post.

[gogaruco]: http://gogaruco.com/

While on my hiatus, some comments trickled in on old blog posts. I do want to go
back and write updated versions of a few of my older articles. But today I want
to talk about Refactoring. I watched
[Therapeutic Refactoring by Katrina Owen][confreaks] the other day and it got me
thinking. While I was at GoGaRuCo I made a point of talking to people, and I
noticed the sly smile on their faces when I told them that, "I love
refactoring." They loved it too. So why don't we do more of it? If it's
therapeutic, if it's exciting and makes our code better, why aren't we always
refactoring?

[confreaks]: http://www.confreaks.com/videos/1071-cascadiaruby2012-therapeutic-refactoring

The reason, I think, is that along with that feeling of making our code awesome,
and coding purely to write code - no feature requests, there is a very real
feeling of [impending doom][refactoring]. What if I pull on a thread that
unravels the system? What if I find out that refactoring the mailer actually
means I need to refactor the entire user object?

[refactoring]: http://i.imgur.com/wGUTG.gif

We've all experienced that elated feeling after watching a great presentation
about refactoring. We head back to our keyboards, determined to make our code
better.  But in reality the systems we encounter are never as simple as the
examples we've just seen. The same tricks don't apply. Our objects aren't easy
to isolate, they aren't easy to test and are often so complex that it hurts to
even think about them.

So what are we to do? Here are my thoughts.

1. Change your mindset. The goal is not to make the system **perfect** and
   **easy** to understand. The goal is to make the system **better** and
   **easier** to understand.
2. It's okay to stop. Start small - even if it's not the final solution. Pick
   one function, a few lines or even one line of code, and work on that first.
   Think about creating a pull request with the changes you've just made. Would
   your peers be able to reason about your changes?
3. Don't be afraid to replace two messes with one mess (of equal and approximate
   size). It's an evolving process and each piece of code does not need to be
   final or pretty, it just needs to make your program better in one way. Take
   another stab at refactoring duplication or complexity once the first step is
   done. Too many times have I been sucked down the rat hole of trying to fix
   all the messy parts at once.
4. Find someone to share the experience with. Pair refactoring is a great way to
   tackle the larger messes.
5. Refactor all the time. When a new feature request comes along, see if you can
   refactor first, in a separate commit. Then implement the feature. The goal
   is to make the feature commit simpler.
6. Demand refactoring. If you don't have the support to refactor then get that
   support or find a company/client that values it. Refactoring makes code
   better. Better code makes better products and better jobs.

I hope that helps. And keep your spirits up. Refactoring isn't easy to do the
first time. Soon enough though, you'll begin to view it as "your feature
request." And it will be a regular part of your awesome day.
