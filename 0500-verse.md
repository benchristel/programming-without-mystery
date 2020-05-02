# Case Study 2: A Browser-Based Programming Environment

In this chapter, I'll talk a bit about the principles I
applied to a slightly larger project, Verse—a JavaScript
programming environment contained in a single
HTML file.

One of the things that makes Verse interesting is that it
incorporates a test framework—the idea is that people using
Verse would test-drive their code. It reduces
TDD to the barest essentials, but in a way that results in
delightfully simple, expressive, and extensible tests. One
of the things I hope you get out of this chapter is an
appreciation for how simple TDD can really be.

## Technique: Avoid Unnecessary Architectural Complications

I initially separated Verse into a library and an editor.
I later undid the separation; I realized that, if I made the
library do everything it needed to do, it would be too
tightly coupled to the editor to be reused. Nothing prevented
me from deferring this architectural decision, and perhaps
extracting a library later if the need arose or it seemed
like the right thing to do. Making the decision early
incurred a cost with no immediate benefit.

Avoid making architectural decisions just because it seems
"cleaner" or like the right thing to do. Think about whether
you can safely defer the decision until you have more
information. Often, you *will* have to make the right
decision upfront because it will be hard to change later.
But this wasn't one of those times.

## Technique: Start with Manual Testing

## Technique: Choose frameworks/libraries that facilitate unit tests

If you start with manual testing, you may find it hard to
unit test later unless you've chosen technologies and
architectures that you know will make that easy.

## Technique: Test To Discover What You're Uncertain About

the first test I wrote for Verse was for the Definer
object. I perceived this to be a fairly tricky state-management
problem and in order for the UX to be good it had to work
flawlessly.

Also: middle-out implementation of forms and export:
68c7b2b495b000be48da55440467bd2c56f6ba6e
248360167d14ba86494c972779ff539f8ccdee88

## Technique: Test data transformation where possible

## Technique: Prioritize Risky Work Higher

017246e53049aa8b6f108117111748fff292a121

## Technique: Coroutines For Synchronous Effect Isolation

allowed extremely fast testing of verse apps "through the UI"

## Technique: Time Is A Message

275efa1253df52b3155bd2496f95b41c55138f74
47eca86b1b00a6cfc8cf0c8ba509368373dbe50f

## Technique: Sleep Is The Best Debugger

e3bb85292c95b92896ca8470e74f45a1749b87bc

## An example of an incorrect test leading to a bug

712dfaa724be256c456d9a060bdc9662e7a01728

## Technique: 300 Millisecond Test Runs

7d996d496d39e07ca92461060a6b4e16d8c50df9
