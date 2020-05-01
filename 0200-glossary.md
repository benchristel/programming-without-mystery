# Glossary

> At the heart of [the debate about the merits of TDD] is confusion over terminology and what different things are.
>
> —[Martin Fowler][1]

[1]: foo

## Test-Driven Development

TDD is not just red-green-refactor. If it were, this book
would be a lot shorter. Rather, I use the term TDD to refer
to a whole suite of practices, mental tools, and techniques
that are enabled and encouraged by the red-green-refactor
cycle.

Some critics of TDD seem to equate it with
red-green-refactor. This conflation is often what leads to
claims like “TDD doesn’t work, and I have a controlled study
to prove it.” What these studies have actually shown is that
the red-green-refactor process by itself doesn’t make a
difference to how good your code ends up.  That’s a very
different claim than “TDD doesn’t work,” and it’s one that
matches my experience perfectly.

Red-green-refactor, by itself, will not improve your work.
Don’t fall prey to the hype. To get any benefit from
TDD, you need to understand all the other things that the
red-green-refactor mindset enables you to do.

Until you’ve seen TDD in action, it’s hard to grok this, and
easy to criticize TDD for its baldfaced ridiculousness. I
once compared TDD to riding a bicycle:

> Can you imagine trying to sell someone a bicycle if they've never seen anyone ride a bicycle before? You can talk till you're blue in the face about gear ratios and brake pads and proper tire inflation, but when your customer gets on the bike they're going to fall off, and then you and they will have words. They will tell you that your contraption is obviously ineffective and probably dangerous. Clearly no one could possibly balance on it, especially not at high speeds. They may concede that one could push along the ground with one's feet while sitting in the saddle— if the pedals were removed. But no way are they buying that bicycle.

When someone says bicycles can’t possibly work, what do you do?

> The sane response to this type of criticism is for the bicycle salesperson to just get on the bike and ride. Once the customer sees someone ride a bike, they will stop insisting that bikes cannot work and start rationally weighing their options.

Often, the thing that convinces people to try TDD is simply
watching an expert practitioner work. When you’ve seen
someone carve clean code out of nothing in a series of
smooth, swift strokes, running the unit tests every five
seconds, it tends to leave an impression. Bob Martin was
convinced to try TDD by two hours of pair programming with
Kent Beck.

Going back to the bike metaphor: red-green-refactor is like
training wheels. It’s the first thing you give someone so
they can get a taste of what TDD is like. But it’s not the
whole picture, not the goal, and not even necessary for an
expert practitioner.

The architect Christopher Alexander, describing his vision
of buildings and towns that arise from human nature itself,
put it this way:

> [T]o become free of all the artificial images of order which distort the nature that is in us, we must first learn a discipline which teaches us the true relationship between ourselves and our surroundings.
>
> Then, once this discipline has done its work, and pricked the bubbles of illusion which we cling to now, we will be ready to give up the discipline, and act as nature does.
>
> This is the timeless way of building: learning the discipline—and shedding it.

## Formal Automated Tests

So far, I have talked about testing only in the abstract.
Now it’s time to get down to brass tacks. The first question
we have to address is: what even is a test?

The answer is more subtle than it might appear. The term
“testing” can be applied extremely broadly, encompassing
everything from QA to automated testing to manually running
programs at the command line. I define testing, in this
general sense, as execution of a piece of software performed
with the goal of finding out what it does.

Formal testing is a subset of testing. A formal test defines
clear expectations for what the software should do, under
what circumstances. Because the expectations are clear, a
formal test has a binary result: software can pass or fail
the test. Generally, only software that has passed all its
formal tests is allowed to ship to production.

Formal testing can be performed manually. This is
traditionally the job of a QA team, though I think modern QA
is focused more on exploratory testing, which is a less
formal style. More on that later.

Automated tests are programs that test other programs.
Automated tests can be formal or informal. An informal
automated test might be a script that invokes a program with
several different inputs and prints the results to the
terminal. It doesn't have a “pass” or “fail” condition; a
human has to inspect the output to see to what degree the
program is working.

A formal automated test, on the other hand, defines
expectations, in code, for what the software being tested
should do. Formal automated tests emit an unambiguous
signal, like an exception or error code, when there’s a
failure. This removes the need for humans to interpret the
output of the software being tested.

When I use the word “test,” it’s almost always shorthand for
“formal automated test”. Most other programmers who do
automated testing also use the word “test” this way.

We can organize the types of tests into a grid:

. | Manual | Automated
--- | ------ | ---------
**Informal** | “Poking around the UI”, “trying it in the REPL”, exploratory testing | Test scripts
**Formal**   | Traditional QA | “Tests”

## Unit Tests

Formal automated tests can be categorized further, into unit tests and system tests.

The term unit test is probably one of the most overloaded, ambiguous terms in software. Everyone seems to mean something different when they say they do “unit testing”, and many a flame war has been fought over which definition is the “right” one.

In this book, I’ll define unit test by taking the set union of the definitions I see in common use. That is, if a bunch of people on the internet and in my social circle say something’s a unit test, I’ll include it in my definition.

To qualify as a unit test, a test must have these three attributes:

It can be run in any order relative to other tests, or by itself. It does not depend on other tests to put the software into the right state, nor does it interfere with any other test.
It calls methods/functions/procedures of the production code directly. To make this easier, unit tests are almost always written in the same language as the production code.
It does not depend on a deployed system. You don’t have to build or run the whole application to run a unit test.

Some people say a unit test has to test only a single class in isolation from all others, or even a single method of a class. I find that definition unhelpfully restrictive. The more extreme form, which posits that a unit test must test only one method, effectively prohibits object-oriented programming, since the only reason to have objects is if the methods of an object can interact in an interesting way. When writing object-oriented code, it is often precisely these interactions between methods that we want to test.

The idea that every production class must have a corresponding unit test is unhelpful in a different way: it makes refactorings that create new, privately-used classes and functions more expensive, since you have to also create tests for them, even if the existing tests completely (though indirectly) cover their behavior. Directly testing classes that are private to a module unnecessarily ossifies their interfaces: if no one outside the module is using that class, why should the tests care if it even exists?

## Test Subjects

## Behavior

## Refactoring

## Routines, Procedures, Functions, and Methods

> TODO: Is this really necessary?

## Metaphors

drive
break
brittle
flaky
cover
against
