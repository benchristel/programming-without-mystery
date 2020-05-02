# Reference

> At the heart of [the debate about the merits of TDD] is confusion over terminology and what different things are.
>
> —[Martin Fowler][FowlerTODO]

## Glossary

### test subject / SUT / production code / implementation / "code"
### refactoring
### behavior
### Unit Tests

Formal automated tests can be categorized further, into unit tests and system tests.

The term unit test is probably one of the most overloaded, ambiguous terms in software. Everyone seems to mean something different when they say they do “unit testing”, and many a flame war has been fought over which definition is the “right” one.

In this book, I’ll define unit test by taking the set union of the definitions I see in common use. That is, if a bunch of people on the internet and in my social circle say something’s a unit test, I’ll include it in my definition.

To qualify as a unit test, a test must have these three attributes:

It can be run in any order relative to other tests, or by itself. It does not depend on other tests to put the software into the right state, nor does it interfere with any other test.
It calls methods/functions/procedures of the production code directly. To make this easier, unit tests are almost always written in the same language as the production code.
It does not depend on a deployed system. You don’t have to build or run the whole application to run a unit test.

Some people say a unit test has to test only a single class in isolation from all others, or even a single method of a class. I find that definition unhelpfully restrictive. The more extreme form, which posits that a unit test must test only one method, effectively prohibits object-oriented programming, since the only reason to have objects is if the methods of an object can interact in an interesting way. When writing object-oriented code, it is often precisely these interactions between methods that we want to test.

The idea that every production class must have a corresponding unit test is unhelpful in a different way: it makes refactorings that create new, privately-used classes and functions more expensive, since you have to also create tests for them, even if the existing tests completely (though indirectly) cover their behavior. Directly testing classes that are private to a module unnecessarily ossifies their interfaces: if no one outside the module is using that class, why should the tests care if it even exists?

### System Tests

### Metaphors

drive
break
brittle
flaky
cover
against
red
green

## How to Start a Project

## Techniques
