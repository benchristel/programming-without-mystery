# Why Test-Drive?

## Five Goals of TDD

This chapter describes five outcomes that TDD enables. You
can think of them as reasons to do TDD.
They are: **a
high-quality product**; **safe, easy change**;
**less waste**;
**enthusiasm**;
and **responsibility and trust**.

### A High-Quality Product

Users want software they can use; product owners want
software they can sell; programmers want software they can
be proud of. Quality is at the nexus of these desires.

But what exactly is quality? Definitions often focus on
conformance to requirements. I think this misses the mark,
though. The concept of requirements presumes that users know
in advance what they want, and can specify their desires as
a list of concrete feature requests. In other words, it
presumes that each user already has a mental model of how
the software should work, and that it is the development
team’s job to produce software that matches this model as
closely as possible.

However, what happens more frequently in actual software
development is that the users do not come to the table with
an exact mental model of how the software should work.
Instead, they start with a partial model and fill in the
details by experimenting with the software and seeing what
it actually does. As a user of software, you’ve no doubt
experienced this: you install a new text editor, chat app,
or photo-editing tool. By playing around with it, you figure
out how it works. If you can figure it out quickly, the
software is high quality. If, on the other hand, the
software is confusing, or does things you don’t expect,
you’ll rate its quality lower.

This judgement of quality is independent of the set of
features in a product. A product’s early releases may not do
everything its users want or theoretically “require”, yet we
can still say that these releases have high quality if the
functionality that is present is easy for the users to
model. When I think of this type of quality, I picture a
certain conceptual coherence. High-quality software is
marked by an absence of disorder.

With that in mind, here is my attempt at defining quality: Quality is what
makes it easy for the user to construct a useful mental model of the software.
A useful mental model is one that enables the user to:

- achieve their goals
- figure out when the software won’t help them achieve their goals

To enable the user in this way, a mental model must predict what the software
will do given a particular sequence of inputs (e.g. button clicks or
keypresses). High-quality software gives us the information we need to make
these predictions accurately. We can then reason from our goal to a (possibly
partial) plan of action, and then carry out the plan. Low-quality software, by
contrast, thwarts our expectations, hides useful information from us, and makes
it harder to formulate a plan of action that will work. Any time you’re using
software and think “why the heck did it do that?” you’re confronting low
quality.

This definition makes it clear that quality depends as much on the user as it
does on the software. For example, Git is a high-quality piece of software for
a user who understands content addressing, Merkle trees, and Unix command line
conventions. It is a low-quality piece of software for the average,
non-tech-savvy person who tries to use it, even if it technically solves their
problem. (Having been both of these users myself, I can personally attest to
this.)

Quality may not even be about the software per se. Cogent, well-organized
documentation can also help users build a mental model, and so is a factor in
the software’s quality.

Software quality can be divided into internal and external quality. External
quality is what the user sees. Internal quality, by contrast, is about the
shape of the code. High internal quality is what makes the code easy for
programmers to mentally model and change. The definition of internal quality is
actually identical to the definition of quality given above, but now the
developers are in the user’s role, and the behaviors they’re modeling are
internal to the program.

Internal quality supports external quality. It’s unlikely (though theoretically
possible) that you’ll have high external quality if you have low internal
quality. Low internal quality means there are more confusing bits of code where
bugs and glitches can hide, and these misbehaviors confound and complicate
users’ mental models.

You can’t ship software faster by sacrificing quality. This is true of both
external and internal quality. One reason for this is that programmers are also
users of the software: during development, they have to build atop existing
code. Often, they have to install the software, run it, debug it, and interpret
its error messages. When quality is low, each of these steps is more difficult
and takes longer. The drag on development imposed by low quality derives
directly from the definition of quality: low-quality software is hard to
maintain and extend precisely because it is hard to model accurately, and when
you change it, it responds in ways you don't expect.

Given the definition of quality above, it’s easy to see how tests contribute to
it: tests describe what the software will do given a particular sequence of
inputs. They thus inevitably make the software’s behavior easier for
programmers to model—the tests function as a kind of documentation. TDD goes a
step further, though: by writing the tests first, programmers have to construct
a mental model of the behavior before the software exists. This makes it more
likely that the model will be comprehensible, since it derives only from the
programmer’s intention and not from some existing piece of code.

Note, however, that the tests are not the model, and cannot substitute for the
model. The model is a complex, unserializable mental construct that lives only
in the programmer’s brain, and can be only imperfectly communicated. Like
orthographic projections of a physical object's design, the tests, code, and
documentation each provide only a partial view of the model. Thus, if the tests drive
the code, then the model must drive the tests.

For more on the importance of mental models in software development, see Peter
Naur’s “Programming as Theory Building”. He uses the word “theory” where I use
“model”, but it’s the same idea. For an in-depth exploration of the concept of
Quality, I suggest Robert Pirsig’s Zen and the Art of Motorcycle Maintenance
and Christopher Alexander’s books The Timeless Way of Building and The Process
of Creating Life.

### Safe, Easy Change

To arrive at high quality, we have to be able to change. Users and business
stakeholders want software to change for all kinds of reasons, and not all of
their desires can be anticipated. To be sure, upfront planning is often
valuable, but the ability to change reactively provides an escape hatch from
situations that could otherwise spell a project’s doom.

Even if all of the desired behaviors of the software can be specified upfront,
the engineering team still must confront the fact that software development is
a process that occurs over time. In other words, software changes simply as a
fact of its production. Since we can’t do everything all at once, nor keep the
entire system in our heads, we have to plan and develop iteratively, lest we be
overwhelmed by complexity.

As John Gall points out in Systemantics,

> A complex system that works is invariably found to have evolved from a simple
system that worked. A complex system designed from scratch never works and
cannot be patched up to make it work. You have to start over, beginning with a
working simple system.

TDD scaffolds this process. TDD lets you begin with something trivially simple
and gradually layer on more and more complexity. The refactoring step keeps the
code as simple as it can be while still displaying the complex external
behavior required of it.

TDD also helps us think about the modularization of our software, warning us
when we try to bite off too much complexity at once, or when the interactions
between modules are too complicated. Sometimes the mere act of writing a test
is enough to spark an idea for a simpler interface design. Of course, there is
no cheaper time to change the interface than before the code exists.

Finally, TDD makes change safer. Test coverage helps us ensure that when we
change software to accommodate a new request from users, we’re changing only
what we intend to: i.e. not introducing bugs.

### Less Waste

In their paper "Software Development Waste", Todd Sedano,
Paul Ralph, and Cécile Péraire describe an empirically-based
taxonomy of software development waste. They identify nine
types of waste that occur on software projects; I have found
that TDD helps reduce six of them:

- **Unnecessarily Complex Solutions.** TDD helps programmers focus on writing
  the simplest implementation that will pass the tests. Once the simple code is
  working, teammates can have an evidence-based conversation about whether
  non-functional improvements, e.g. for maintainability, performance, or
  security, need to be incorporated.
- **Extraneous Cognitive Load.** Tests form a self-verifying to-do list:
  failing tests represent work that still needs to be done; passing tests are a
  list of behaviors that are already implemented. Having this list reduces the
  amount of state that programmers need to keep in their head as they work. Tests
  also allow programmers to refactor safely, keeping code simple and readable as
  they work.
- **Knowledge Loss.** Inevitably, programmers leave the team. If they are the
  sole possessors of specialized knowledge that the team needs, their departure
  may be disastrous for the other team members. Tests help document their code
  and protect it from bugs, increasing the chances that the rest of the team will
  be able to continue where they left off.
- **Waiting.** Time spent waiting for tests to run is waste. TDD enables more
  of the code’s behavior to be checked by fast-running unit tests. This can
  reduce test runtimes by several orders of magnitude over a system-test-heavy
  approach.
- **Ineffective Communication.** The unit tests produced by TDD constitute a
  kind of executable developer documentation, which describes exactly what each
  module does. Since the tests are kept passing at all times, the documentation
  they provide never goes out of date.
- **Rework.** Code that has to be thrown away and rewritten is waste. TDD
  encourages programmers to minimize rework by dividing code into small,
  single-purpose modules that can be kept and reused even when the code around
  them is in flux. We can compare this to the use of replaceable parts in
  mechanical engineering: instead of replacing a whole machine, you replace just
  the part that doesn’t work.

Sedano et al. also identify three types of waste that TDD can’t help with:

- **Building the Wrong Feature or Product.** Tests can’t tell you if what
  you’re building is valuable (unless they’re acceptance tests written by the
  users)—all they can do is catch programming mistakes.
- **Mismanaging the Backlog.** This type of waste relates to ineffective
  communication between the engineering team and the product manager. It
  manifests as the team not having enough to do or working on low-value things.
  Since TDD is a practice that’s internal to the engineering team, it can’t
  address this type of waste.
- **Psychological Distress.** While my personal experience has been that TDD
  makes me happier, it doesn’t directly counter the sources of psychological
  distress Sedano et al. describe, which include deadline pressure and
  interpersonal conflict.

### Enthusiasm

A failing test sets a clear short-term **goal**; seeing all
the tests pass provides a sense of **accomplishment**. This
loop of goal-setting and accomplishment keeps me focused and
motivated for a full day of work.

Test-driven code also provides a feeling of **mastery**.
When I test-drive, I usually start with the error cases,
trivial cases, and edge cases (see the chapter “Thorns
Around the Gold”). This means that by the time I’ve gotten
the main behavior working, I feel I’ve fully nailed down the
code, and I can move on to the next task with no nagging
worries or regrets. When I create new features that depend
on test-driven code, I can be confident that I’m
building on a solid foundation.

For more on the value of mastery, see Sal Khan’s TED talk
“Let’s Teach for Mastery—Not Test Scores”.

### Responsibility and Trust

Teams that test-drive can develop a culture that balances
**autonomy** (any team member can work on any code) with
**shared ownership** and **responsibility**. For example, my
teammate and I might agree that I can modify a library she
wrote, as long as I don’t break any of her tests and I add
tests of my own for new behavior. The existing tests
document her original intent, and catch the bugs I otherwise
might introduce.

Simply seeing tests and clean code when I delve into an
unfamiliar part of the codebase renews my sense that someone
cares about this code. That makes me care more about not
messing it up.

Since failing tests provide an objective way to tell that
something’s wrong, they create an opportunity to deliver
constructive feedback in a way that doesn’t feel like a
personal attack. On many teams, the norm is that the
continuous integration monitor should stay green. If I break
a test and the monitor goes red, a simple "hey, build's
red!" from a teammate is all the feedback I need to correct
my mistake. Over time, this blameless mistake-catching can
build up a sense of safety and trust on the team.

Delivering a high-quality product with less waste through
incremental, safe, easy change helps build trust between the
engineering team and other stakeholders. Once programmers
have established a reputation for reliably delivering
high-quality results, other people in the organization start
listening to them and trusting their judgement. Management
becomes more hands-off. Programmers' warnings of schedule
slips, unrealistic feature requests, and possible bugs are
taken more seriously. In other words, the programmers are
treated as if they know what they’re doing—because they do.

## What Work Is Like With Effective TDD

This chapter describes the high-level patterns that can
arise among development teams using TDD to pursue
the five goals listed in Part One. When I join a new team,
these are the things I look for. They are the vital signs of
a healthy testing culture.

It's possible to "do TDD" in the sense of red-green-refactor
without enabling these patterns or realizing their benefits.
When these patterns are absent or hollow, it often points to
deficiencies in the team’s approach to testing and design.
If I find myself on such a team, I'll try to **gradually**
correct these deficiencies, enabling the patterns to emerge
when they are natural and necessary.

Incorporating TDD into a team that hasn't been doing it
can take months or years of slow, steady change. It can
feel like a lot of effort. But I
honestly believe the benefits are worth it.
By writing this chapter, I hope to show you what,
concretely, your team can look like, if you and other
members of your organization diligently pursue the five
goals using the techniques demonstrated in the case studies
to follow.

### Programmers Write Tests

A necessity of TDD is that the programmers writing
the production code must also write the tests. There's
simply no other way for the tests to be written in tandem
with the code. That programmers write tests is already a
given in most modern organizations, but I thought I'd call
it out explicitly.

### The Tests Are Fast

### Everyone Can Run The Tests

### The Tests Run Frequently

### The Tests Catch Mistakes

### Feedback Is Effortless

### Review Happens Continuously

Most teams do code review: programmers read each others' code to catch mistakes and suggest improvements.
But TDD, because it externalizes the thought process of designing and writing code, allows programmers
to critique each others' thought processes and empirical methods during pairing sessions. The "reviewer"
can see how each test the "author" writes motivates a change to the code, and can verify that refactorings
are indeed behavior-preserving. This results in higher confidence in the code (not to mention faster review)
than if they just read the end result.

### Tests are a Collaboration Tool

tests start conversations

e.g. ping-pong pairing

e.g. cross-team contract testing

### Vacations are Not Show-Stoppers

### Dependencies are Visible and Understood

### The Effects of Code Are Obvious From Its Interface

### Contracts Are Formalized

### Technical Constraints Align With Social Constraints

### Much of the Code Never Has to Change

### Status Is Evident

### The Code Has Natural Symmetry

### Important Differences Stand Out

### Internal Design Is Emergent

### Bugs Are Easy to Fix

### Code Can Be Repurposed

e.g. to diagnose production issues.

"write code that you trust enough to call during a
production outage"

### There Is Constant, Gradual Improvement

Example: team reducing duplication and improving code
organization

### The Right Thing to Do Becomes Obvious
