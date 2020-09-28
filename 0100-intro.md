# Introduction

This book collects a set of ideas, principles, and
techniques which I collectively term
*Empirical Software Development* (ESD). ESD provides
scientific approaches to solving the following problems:

- Deciding what software to build
- Finding defects in existing software
- Deciding what modifications to make to existing software
- Designing the details of user interfaces
- Creating maintainable, bug-free code

## ESD, Abstractly

ESD applies a single, fundamental process, closely mirroring the scientific method, to each problem that arises in software development.

- Observe the system.
- Formulate a theory about the system, in the form of a set
  of rules describing its behavior.
- Try to prove that the theory is wrong, by finding 
  specific situations that break the rules.
- When you find such a situation, revise the theory so
  that it accommodates all observations to date and is as 
  simple and general as possible.
- Repeat until a desired level of confidence in the theory
  is reached.

The following auxiliary principles are also observed:

- Keep a record of the experiments that falsified past
  theories, as they may also be useful in falsifying
  future ones.
- Keep the time and effort needed to falsify a theory to an
  absolute minimum.

That is everything you need to know about ESD, in the abstract. How best to apply these rules to specific cases
is, unfortunately, not always obvious. The rest of this
book is thus dedicated to fleshing out the technical and
practical specifics.

## Test-driven development

In large part, this book is about how to apply the programming technique
known as **test-driven development** (TDD) on complex,
real-world software projects. TDD is deceptively simple.
It’s easy to do, hard to do well. But the reward for doing
it well—clean, defect-free code—is worth it.

What is TDD? The core of the technique is often summarized
as **red, green, refactor**—a three-step cycle that’s
repeated over and over for every programming task:

- **Red:** Write an automated test that describes what you want the code to do. Watch it fail.
- **Green:** Write the code to make the test pass.
- **Refactor:** incrementally restructure the code to remove duplication and simplify the whole system.

The three-step process paints a blurry picture, though. What
does it mean for a test to "describe what you want the code
to do"? What does it mean to "simplify the whole system" by
refactoring?

Many answers to these questions have been proposed, but not
all work well in practice. My own journey toward discovering
good answers has been circuitous, full of backtracking and
hard-learned lessons.

This is the book I wish I’d had when I started out. It
describes the concepts and techniques I’ve learned over the
last decade of my programming journey and presents them in a
way that I hope will be accessible to people who’ve never
done TDD before. Along the way, it delves deep into the
philosophies of testing and program design, examines the
mechanics of coding, and traces foundational concepts to
their source. I believe this comprehensive treatment is
necessary, because *TDD changes everything*. There is no
part of software development that is not radically altered
by it, to the benefit of the whole.

Test-driven development has transformed the way I write
software. I’ve written this book in the hope that it can do
the same for you.

## What's In This Book?

This book has seven main sections.

- **Why Test-Drive?** Enumerates the goals that TDD
  helps software teams achieve, and presents a vision of
  what work can be like when TDD is done well. These
  descriptions are grounded in my experiences on many different teams that
  have successfully integrated TDD into their workflows.
- **Difficulties.** Describes all the ways in which TDD can go
  awry. It provides important context for understanding the
  warnings and caveats in later sections.
- **A Case Study.** Shows how I've applied TDD to one of my own projects. It goes into detail with code
  examples and explanations of the techniques I applied.
- **Principles.** Distills general principles
  from the case study and illustrates them with specific
  examples.
- **Reference.** Summarizes the ideas and
  techniques from the earlier chapters. Essentially, this is
  a recapitulation of the main points of the book, optimized
  for easy access.
- **Coda.** Offers a holistic view of the book's subject
  matter, lest we miss the forest for the trees.
- **Further Resources.** Lists the books, papers, and
  videos that have helped and inspired me.

## Who Should Read This Book?

Software development is a vast field, and although I’ve been
programming for twenty-two years I feel I have barely
scratched its surface. Accordingly, I can only write about
how to solve the types of problems I have encountered in my
work, and I'll assume that you are solving similar problems.
If your work differs significantly from these assumptions,
my advice may not be good. Proceed with caution.

I'll assume you're doing application programming in a
multi-paradigm language—one with support for
object-oriented, functional, and procedural styles. For
example, any of these:

- Java
- JavaScript
- Python
- Ruby
- Go
- Rust
- Kotlin
- Scala
- C++

Other similar languages will probably work too, as long as
they have either a fast compiler (like Go), an incremental
compiler (like Java), a dynamic interpreter (like Python),
or some combination of these (like JavaScript). Slow
compilation makes the techniques in this book much more
tedious to apply.

## A Plea for Pragmatism

The guidelines in this book aren’t rules to follow
dogmatically. They’re just techniques that I’ve found work
well in many situations. Think of them as "sane defaults" or
"first things to try." They’re not set in stone, and you’ll
probably find ways to improve on them.

However, if it’s not obvious at first how to apply the
techniques to your situation, I urge you not to give up.
They work in more situations than people often give them
credit for. When you feel stuck, the insight you’re missing
will usually not be about testing, but design. Good
resources on the topic of design include:

- Sandi Metz and Katrina Owen: _99 Bottles of OOP_ (e-book)
- Eric Freeman, Bert Bates, Kathy Sierra, and Elisabeth Robson: _Head First Design Patterns_ (book)
- Eric S. Raymond: _The Art of Unix Programming_. (html book) http://www.catb.org/esr/writings/taoup/html/

## Learn from the experts

The quickest way to grasp the basics of TDD is to watch an
expert do it. I recommend these videos by Bob Martin:

- https://www.youtube.com/watch?v=SVRiktFlWxI&t=7h30m44s
- https://www.youtube.com/watch?v=qkblc5WRn-U

If you’d rather learn the basics from a book, Kent Beck’s
_Test-Driven Development by Example_ is the foundational
text. _99 Bottles of OOP_ by Sandi Metz and Katrina Owen is
also a fantastic resource, a treasure trove of insights
about design and testing.

I recommend practicing TDD on your own before bringing it to
work. To start you off, https://exercism.io has TDD coding
exercises in over 40 programming languages. The tests are
already written for you, so you get a gentle introduction to
testing strategies.

There’s much more to the TDD approach than meets the eye in
these examples. Notably absent is any discussion of how TDD
shapes the development of large, complex software systems.
The reason this book exists is to fill in those gaps.
