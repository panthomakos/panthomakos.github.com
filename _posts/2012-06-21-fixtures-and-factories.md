---
layout: post
title: Fixtures and Factories
category: Best Practices
tags: [ruby, rails, tdd]
---

I recently converted a test suite that only used factories to a hybrid that uses
a combination of factories and fixtures. While I really like the simplicity and
cleanliness of factories, fixtures are simply unbeatable in some cases. I had
never really used both together in the same project, so I wanted to outline the
pros and cons of each, as well as how they work together.

### Loading and Creating Data (Performance)

If you're loading all of your fixtures at the beginning of every test you're
missing out on one of their major benefits - loading a lot of data only once.
Instead, switch to loading all of your fixtures at the beginning of the entire
test suite, and then using transactions around each test for cleanup. That way
the base fixture data hangs around and each test takes care of cleaning up its
own changes. Since generating data can be more expensive than saving it to the
database, this simple change can result in dramatic performance gains.

When using factories you have the benefit of creating only the data you want.
You may face some degraded performance caused by create operations, since you'll
have to process all those pesky callbacks, but you will benefit from having more
isolated tests and data.

### Sharing and Customizing Data (Maintainability)

Fixtures often make customizations to data more complex. It's certainly a lot
harder to change fixtures because other tests depend on that very same data.
Certainly a slew of update statements is going to offset some of that
performance gain of having data pre-loaded as well.

I often find that fixture based tests end up doing a lot of pre-assertions to
ensure that data is in the correct state. If that's the case in your tests it's
probably worth asking yourself if you could be using factories instead.

One way to combat the fixture modification data is to combine fixtures and
factories. For instance, if you need to parse a large XML file and then create
a model from that data, do the parsing ahead of time, store it as a fixture, and
then use that model as a base for your factories.

Factories should generally be an MVO, or minimally-viable-object. Data sharing
becomes a non-issue because each object adds only what it needs without worrying
about conflicting with other objects or tests.

### Proximity of Data and Assertions (Ease of Use)

In the fixture approach it's often harder to find the data related to a test
because it's usually contained in another file. Sometimes this is unavoidable,
even with factories, but it's more often the case that fixture data lives in
some other file. If you are making an assertion about data, it's best to have
that data near the assertion.

In the factory approach the relevant data declarations and modifications are
all contained in the same test file which makes the test cases easier to read
and follow.

### Rule of Thumb

No approach is ever 100% better. In fact you'll probably see the greatest
performance gains and have the most fun by using both factories and fixtures.
It feels great to use the right tool for the job. If you use transactions to
clean your tests then the two approaches work very well together. Fixtures will
load data before the entire suite, and then each test will automatically
rollback any changes to fixture data, and remove any factories it created.

As a rule of thumb, if data is expensive to create, it's probably better to use
fixtures. If data requires many small customizations, it's probably better to
use factories.
