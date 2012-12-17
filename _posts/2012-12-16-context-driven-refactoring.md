---
layout: post
title: Context Driven Refactoring
category: Refactoring
tags: [ruby, refactoring]
---

I got excited about this refactoring website called [refactr.it][refactr]. Users
post refactoring questions and vote on the best answers. The website is kind of
a simplified version of the [Programmers Stack Exchange][pse]. But as I explored
the ruby section, which is definitely still in it's infancy, I was a little
disappointed.

[refactr]: http://www.refactr.it
[pse]: http://programmers.stackexchange.com

Here is an example of a question and solution that was titled
"[Simplifying Nested If Statements][example]".

[example]: http://www.refactr.it/problems/50bab8def2f2231000000001

## Question Code

{% highlight ruby %}
@problems =
    if @tags.blank?
        unless @language.nil?
            Problem.where(language: @language).desc(@sort).page(@page)
        else
            Problem.desc(@sort).page(@page)
        end
    else
        unless @language.nil?
            Problem.in(tags: @tags).where(language: @language).desc(@sort).page(@page)
        else
            Problem.in(tags: @tags).desc(@sort).page(@page)
        end
    end
{% endhighlight %}

## Solution Code

{% highlight ruby %}
@problems = Problem
@problems = @problems.in(tags: @tags) unless @tags.blank?
@problems = @problems.where(language: @language) unless @language.nil?
@problems = @problems.desc(@sort).page(@page)
{% endhighlight %}

The solution is definitely a start. In fact it "simplifies the nested if
statement" perfectly. And I would be a much happier programmer if I happened
upon the solution code than I would be if I came across the question code. My
disappointment is that the refactoring lacks context. It's just an example of a
refactoring formula:

{% highlight ruby %}
# Nested

if A
  if B
    Z.X(A).Y(B)
  else
    Z.X(A)
  end
else
  if B
    Z.Y(B)
  else
    Z
  end
end

# Not Nested

chain = Z
chain = chain.X(A) if A
chain = chain.Y(B) if B
{% endhighlight %}

The solution is "correct", and it's a valuable technique, but it misses the
mark. Refactoring often falls short **because** people blindly apply a selection
of these transformations and then call it a day. **Refactoring isn't just about
rewriting code.** Let's take a look at a couple of questions that might help us
devise a better strategy for approaching this problem.

## Question One: Why should I refactor this code?

Refactoring is fun, but if the code is a mess, and you don't need to touch it,
[leave it alone][mess]. As Sandi Metz explains, if you can't see the mess, it
doesn't exist. **Refactoring should always happen for a reason.** In this case
there are multiple potential reasons we might need to refactor. But leaving that
reasoning out of the discussion can lead to a misguided solution.

[mess]: http://confreaks.com/videos/1115-gogaruco2012-go-ahead-make-a-mess

Do I need to add a new search parameter? Then I'm probably in need of a solution
that reduces complexity.

Am I replicating this code somewhere else? Then I'm probably in need of a
solution that DRYs up my existing implementation.

## Step One: Establish a context for the refactoring.

Let's start with, "I need to add a new search parameter." This helps us
establish that the code is too complex, and that we need to address this issue
because we need to change the code.

## Question Two: How am I going to test this code?

Imagine, first without using a database, how you would go about testing the
question code, or even the solution code for that matter. You would definitely
have to provide stubs for `#in`, `#where`, `#sort` and `#page`. You would have
to test conditionals for `@language`, `@tags`, and your new search parameter.
And all those tests have to combined with the rest of the controller tests for
the search action.

Even if you allowed yourself use of the database, you would still have to
construct objects for each of the test scenarios and verify that the
conditionals were working properly.

That's a lot of stuff (otherwise know as responsibilities). The "always write
tests" mantra isn't just meant to increase code confidence, it helps illuminate
design flaws and unwanted complexity.

## Step Two: Read/Write Tests.

Assuming the tests don't already exist, write them. Even though they are brutal
to write after the fact, you need to have test coverage before you start moving
code around.

## Question Three: What did the tests teach us?

Probably that we have a "Fat Controller". You've probably heard of "Fat Models,
Skinny Controllers", but this heuristic does not really mean Fat
`ActiveRecord::Base` classes and Skinny `ApplicationController` classes. It
means that objects responsible for connecting business logic and views, should
do only that. It's the "Single Responsibility Principle" in Rails terms.

The reason we should strive for single responsibility objects is that they are
easier to extend and test. In this case we've seen how difficult it is to test
the code (in both the question and the solution) which indicates that the
proposed solution should probably not be our first refactoring step.

## Step Three: Refactor Your Code

Because the tests indicated an unnecessary complexity, we know that we should
probably introduce a new object this is responsible for handling some of the
logic, without involving the controller.

{% highlight ruby %}
@problems = ProblemSearch.new(@language, @tags, @sort, @page).find
{% endhighlight %}

So what goes into `ProblemSearch#find`? Probably the original solution, but
at this point it really doesn't matter because it's completely isolated. The
original non-nested code is definitely simpler and easier to extend than the
nested code, but that's the transformation part of refactoring, not the reason
we are refactoring in the first place.

You should always simplify your programming constructs so that they are more
intention revealing, but the meat of this refactoring is really in the domain
modelling and the tests. The original code had too many responsibilities, it
was hard to test, and it was hard to extend.

When we needed to add a new search parameter and tried writing tests, we
realized how complex that was going to be and identified the need for a
refactoring. Allowing that context to guide us, resulted in an intention
revealing, testable and extensible solution.
