# Case Study: An Immutable Key-Value Store

In this chapter, I'll be exploring the implementation of
an immutable key-value store with an HTTP API. Here,
*immutable* means that overwritten values are
preserved, and can be looked up by revision number later.

Implementing a key-value store might seem trivial. But the
requirement to make it *immutable* opens the door to a world
of intriguing complexity. What's the most space-efficient
way to store and access different versions of values?
How can we avoid storing tons of duplicate data when someone
changes a few bytes in the middle of a multi-megabyte file?
These questions provide an interesting backdrop against
which the scene of this case study will be set.

The code for this project is in Ruby. Ruby is an interpreted
object-oriented language, vaguely similar to Smalltalk,
Python, and Perl. I've rephrased some of the example code
to avoid Rubyisms that would take a lot of explanation and
distract from the main point. If you know Python or Java,
you should be able to follow along.

## Technique: Readme-Driven Development

Okay, it's day one, and we have no code. We're building an
immutable key-value store with an HTTP interface. What
exactly does that look like?

One technique I like to use at this early stage of
development is to write an aspirational readme file that
demonstrates what I want the end product to look and feel
like. Writing documentation for a product that doesn't yet
exist might feel backwards, but it can be a fun, cheap way
of spitballing ideas, getting feedback from users and
business stakeholders, and reaching alignment as a team on
what exactly we're going to build.

This readme-writing stage of the project is a time to dream
big—but perhaps not too big. It can be easy to get carried away
designing elaborate features and later realize there's no
way to implement them. Be sure to communicate to
people who see the readme that it's not set in stone. Rather, it's a
starting point for negotiations about the cost and value of
features.

Below is the complete text of the initial README.md file I
wrote for the key-value store project. Since the store is
a container that preserves values you put in it, I titled
the project `fridge`.

If this were a readme for a team project, I'd probably
want to state at the top of the file that it's just a draft,
and subject to change. However, this was just a personal
project, so there was no need to do that.

> # Fridge
>
> Fridge is an immutable key-value store. It usually stores values as files on
disk, but the storage layer can be swapped out for e.g. a relational database
or an in-memory store.
>
> ## Revisions
>
> A Fridge database is versioned. Any change to a value creates a new,
incrementing revision number, which is returned from the API call.
>
> You can ask for the value corresponding to a key as of a particular
revision. Details of the API are below.
>
> ## HTTP API
>
> ### Setting a key-value pair
>
> ```
> PUT /values/foo HTTP/1.1
> Content-Type: text/plain
>
> Hello, world!
> ```
>
> Response:
>
> ```json
> {
>   "revision": 1,
>   "timestamp": "2020-01-01 17:45.678Z"
> }
> ```
>
> ### Getting a key
>
> ```
> GET /values/foo HTTP/1.1
> ```
>
> Response:
>
> ```
> Hello, world!
> ```
>
> ### Getting the value of a key at a previous revision
>
> ```
> GET /values/foo?revision=1
> ```
>
> Response:
>
> ```
> Hello, world!
> ```
>
> ### Getting the value of a key at a revision when it did not exist
>
> ```
> GET /values/foo?revision=0
> ```
>
> Response:
>
> ```
> 404 Not Found
> ```
>
> ### Getting the value of a key at a timestamp
>
> ```
> GET /values/foo?time=2020-01-01
> ```
>
> ### Getting the revision history
>
> ```
> GET /revisions?limit=50&mostRecent=100
> ```
>
> Filtering by date:
>
> ```
> GET /revisions?limit=50&before=2020-01-01
> ```
>
> Response:
>
> ```json
> {
>   "revisions": [
>     {
>       "id": 1,
>       "timestamp": "2020-01-01 17:45:01.789Z",
>       "change": "create",
>       "key": "foo"
>     }
>   ]
> }
> ```
>
> ### Pruning old revisions
>
> ```
> DELETE /revisions/10
> ```
>
> This deletes revision 10 *and every earlier revision*.

Already, that's a lot of functionality, and we're not going
to implement it all in this case study. However,
we *will* have to think about how to architect the storage
layer to enable features like
pruning of old revisions to be implemented efficiently.

## Technique: Start With a Walking Skeleton

A _walking skeleton_ is a codebase that has no functionality
yet, but is nevertheless *runnable*. It has a passing test suite
(though it doesn't yet test anything real) and the
production code can be deployed or run locally (though it
just prints "hello world" or something similarly trivial).

The purpose of a walking skeleton is to provide a solid
framework on which we'll hang our program's functionality.
It gets all the infrastructure out of the way, so that in
later steps of the process we can focus entirely on what
we want to build.

There are many frameworks and project templates that
essentially give you a walking skeleton out of the box.
These often come with elaborate features: built-in test
frameworks, linters, hot-reloaders that swap new code
into a running application, and debugging tools.

As convenient as these templates are, they come with a
hidden cost: *on day one of your project, before you've even
written a line of code, you will not understand how your
system works*. My experience of using such templates is
that their fancy features often break as the project gets
more complex, and I don't know enough to fix them. That puts
me in a pickle: do I invest an unknown amount of time in
figuring out what's going on and mending the system, or do
I simply forge ahead with a half-working development
environment?

Often, I feel pressure to simply carry on in a half-working
state. This decision practically dooms the development
environment to keep on degrading until the framework that
was once a convenience becomes a barnacle: a parasite that
slows the whole project down.

I avoid this dilemma by keeping the system simple from the
very beginning, and adding complexity only where it's
required. You might think this would eventually lead to me
reimplementing all the stuff I would have gotten
"for free" with a project template, but in fact, a lot
of that stuff just isn't necessary. When you have simple code with fast,
reliable tests, the need for many other development tools and conveniences
melts away.

When creating a walking skeleton, I usually create a file
structure something like this:

```
Makefile
README.md
dependencies.txt
src/
  main.js
test/
  nothing_test.js
```

That is, I include these components:

- A **build system** that simplifies compiling the production
  code and running the tests. I like to use `make` as a
  wrapper around whatever language-specific tools I'm using,
  since it lets me have a consistent development workflow
  for all my projects. Simple shell scripts, e.g. `./build`
  and `./test`, can also make a good development interface.
- A **README.md** file that documents the project's purpose,
  interface, and/or development workflow.
- A specification of the project's **dependencies**, to be
  used by the **package manager**. This generally has a
  language-specific name; I've used the generic `dependencies.txt`
  in the illustration above.
- A directory for the **source code**, with a "hello world"
  program.
- A directory for the **test code**, with a test that always
  passes (e.g. by making no assertions). Note that some languages and
  frameworks, e.g. Go or Angular 2, put tests and code in
  the same directory tree. Either convention is fine; do
  what makes sense for your language/framework.

Once I have all of this set up, I can:

- Run one command to install the project's dependencies.
- Run `make test` to run the one passing test.
- Run `make run` to build and run the
  "hello world" program.

## The Walking Skeleton for Fridge

Here's the structure of the walking skeleton I made for
Fridge:

```
.ruby-version
Gemfile
Gemfile.lock
Makefile
README.md
server.rb
spec/
  unit/
    nothing_spec.rb
  functional/
    hello_world_spec.rb
```

You'll notice that this follows the outline of the generic
walking skeleton illustration above, but the
details are specific to Ruby and web servers:

- **.ruby-version** specifies the version of the Ruby
  language that must be used with this project. The file's
  name and content are meaningful to the program `rbenv`,
  which helps you install multiple ruby versions and switch
  between them.
- The **Gemfile** lists the packages (Ruby calls them _gems_)
  on which this project depends. The initial version lists
  two dependencies: `sinatra` (a web server framework), and
  `rspec` (a test framework).
- **Gemfile.lock** specifies the exact version of each gem
  that's used by the project. The Gemfile typically
  specifies somewhat loose constraints on gem versions, but
  for the sake of consistency and prevention of "works on my
  machine" bugs, it's desirable to lock down the exact
  versions of each gem that are initially installed.
- The **Makefile** specifies an interface for interacting
  with other tools during development. Here's the text of
  my Makefile:

  ```Makefile
  .PHONY: test
  test: unit functional

  .PHONY: unit
  unit:
  	bundle exec rspec spec/unit

  .PHONY: functional
  functional:
  	bundle exec rspec spec/functional

  .PHONY: run
  run:
  	ruby server.rb
  ```

  If you don't know `make`, here's the gist: The line

  ```Makefile
  test: unit functional
  ```

  says, "to run `make test`, run `make unit` and `make functional`".

  The paragraph

  ```Makefile
  unit:
      bundle exec rspec spec/unit
  ```

  says "to run `make unit`, run the shell command `bundle exec rspec spec/unit`". `bundle exec` and `rspec` are Ruby-specific
  incantations, unrelated to `make`.

  The `.PHONY` lines perhaps require some explanation.
  Make is designed around the concept of *targets*. Its
  assumption is that when you run a command like `make foo`,
  you're trying to generate a file called `foo`, and the
  Makefile should describe how to generate that file. A
  consequence of this is that *`make` often won't run the
  script associated with a target if the file named by the
  target already exists.* `make` tries to avoid doing
  unnecessary work.

  That's not really what we want, though: when we run
  `make unit` we want the unit tests to run even if there's
  happens to be a file named `unit` in our project. When
  using make targets in this way, it's good practice to
  include the `.PHONY` lines, which tell `make` that you
  don't mean for the target names to refer to files.

  I'm applying a tiny bit of foresight here in splitting
  the unit tests and functional tests into separate `make` targets.
  Unit tests are fast and functional
  tests tend to be slow, and I predict that I'll often want
  to run just the unit tests to get feedback quickly as I'm
  coding. If you don't know the difference between unit and
  functional tests—I'll get to that in a bit.

- **server.rb** is the seed of our key-value store program.
  Using the `sinatra` framework, it responds to every `GET`
request with a `200 OK` response code and the text "Hello,
world!".

  ```ruby
  require "sinatra"

  get "*" do
    return 200, "Hello, world!"
  end
  ```
- Next, we have the **spec/** directory, which contains our
  tests. The RSpec testing library calls
  tests "specs", as in *specifications of behavior*.

  `nothing_spec.rb` makes no assertions, and always passes.
  Here's what it looks like:

  ```ruby
  describe "nothing" do
    it "does nothing" do
    end
  end
  ```

  This is, unfortunately, a pretty inscrutable example of
  RSpec's syntax. A more typical spec would look like this:

  ```ruby
  describe "a Stack" do
    it "starts out empty" do
      stack = Stack.new
      expect(stack.empty?).to be true
    end

    it "is not empty after you push one item" do
      stack = Stack.new
      stack.push 1
      expect(stack.empty?).to be false
    end

    # ...
  end
  ```

  That is, a spec *`describe`s* a particular module of code
  or class of object. It lists *examples* of that
  code's behavior, using an English-y syntax: "it does X",
  "it does Y".

  One thing to note about Ruby syntax is that the
  parentheses around method arguments are often left out.
  That is,

  ```ruby
  foo.bar(1)
  ```

  is equivalent to

  ```ruby
  foo.bar 1
  ```

  Both of them invoke a method `bar` on the object `foo`,
  passing `1` as an argument.

  The `do ... end` syntax is called a *block* in Ruby. For
  now, you can think of a block as an anonymous function.
  The syntax `describe "something" do ... end` calls the
  method `describe` with a string argument and a block argument.
  The block defers execution of the code inside it for
  later invocation by the test framework.

  With that background knowledge, we can attempt to read the
  functional test, `hello_world_spec.rb`:

  ```ruby
  require "net/http"
  require "uri"

  describe "Fridge" do
    before :all do
      @server_pid = fork do
        exec "make", "run"
      end
      sleep 2
    end

    after :all do
      Process.kill "TERM", @server_pid
      Process.wait @server_pid
    end

    it "responds to a request" do
      expect(Net::HTTP.get(URI("http://localhost:4567")))
        .to eq "Hello, world!"
    end
  end
  ```

  Most of the complexity of this test is setup and teardown: starting the
  server process by running `make`, and killing it when we're done. In
  between the `before :all` and `after :all` blocks, the `it` block runs,
  asserting that a `GET` request to the server returns `Hello, world!`.

  You don't have to understand all the details of how this test works. The important thing to know is that it's starting a real webserver and making a real HTTP request to it. This test doesn't even need to be in Ruby. I could have written it in Bash or Python, though Ruby is a perfectly convenient choice.

## Terms: Unit Tests and Functional Tests

In the previous section, I used the terms *unit test* and *functional test* without defining them. I'll rectify that now.

### Unit Tests

"Unit test" is probably one of the most overloaded, ambiguous terms in
software. Everyone seems to mean something different when they say they do
“unit testing”, and many a flame war has been fought over which definition is
the “right” one.

In this book, I’ll define what a unit test is by taking the set union of the
definitions I see in common use. That is, if a bunch of people on the internet
and in my social circle say something’s a unit test, I’ll include it in my
definition.

To qualify as a unit test, a test must have these three attributes:

- It can be run in any order relative to other tests, or by itself. It does not
  depend on other tests to put the software into the right state, nor does it
  interfere with any other test.
- It calls methods/functions/procedures of the production code directly. To
  make this easier, unit tests are almost always written in the same language as
  the production code.
- It does not depend on a deployed system. You don’t have to build or run the
  whole application to run a unit test.

Some people say a unit test has to test only a single class in isolation from
all others, or even a single method of a class. I find that definition
unhelpfully restrictive. The more extreme form, which posits that a unit test
must test only one method, effectively prohibits object-oriented programming,
since the only reason to have objects is if the methods of an object can
interact in an interesting way. When writing object-oriented code, it is often
precisely these interactions between methods that we want to test.

The idea that every production class must have a corresponding unit test is
unhelpful in a different way: it makes refactorings that create new,
privately-used classes and functions more expensive, since you have to also
create tests for them, even if the existing tests completely (though
indirectly) cover their behavior. Directly testing classes that are private to
a module unnecessarily ossifies their interfaces: if no one outside the module
is using that class, why should the tests care if it even exists?

### Functional Tests

*Functional tests* test the behavior of the entire application. They're a type of *system test*.

I chose the preceding definition of unit test in part because it contrasts
cleanly with *system test*. Unlike unit tests, system tests interact with a
deployed application. Since they don’t directly call methods or functions in
the application code, they may be written in a completely different programming
language.

- There are many reasons to write a system test. Here are some:
  to test that specific features are working (functional testing)
- to test that specific use-cases or workflows work the way the stakeholders
  wanted (acceptance testing)
- to test that the components that make up the system (e.g. databases, web
  services, and operating systems) interoperate (integration testing)
- to measure how well the system handles large amounts of work (load testing)
  or to find out how much load has to be applied before it completely breaks
  (stress testing)
- to verify that the system can be upgraded or downgraded with no data loss
  (migration testing)
- to verify that the system can come back online after a crash, with no data
  loss (recovery testing)

While your application likely needs some system testing, it’s not the focus of
this book. While some ideas from TDD can be fruitfully applied to system
testing, the red-green-refactor cycle can’t be. System tests are simply too
slow to fit into the TDD rhythm.

Many teams overdo system testing. Functional tests, in particular, are
seductive because they are the most obvious way to test a program’s features.
However, they tend to be slow to run and expensive to maintain.

Over-reliance on functional testing is often a sign that programmers don’t
understand or trust the software’s architecture—the way the pieces fit
together—and thus don’t trust the unit tests to tell them if something’s
broken. The solution is internal quality, not more functional testing. In a
well-architected application whose code affords mental modeling, most
functional tests can be replaced by unit tests. I generally use a combination
of unit testing, manual testing through the UI, and a robust mental model of
the program’s structure to convince myself that a feature works.

Of course, this nonchalant attitude towards functional testing only makes sense
in the context of a rigorous unit-testing discipline. In later chapters, I’ll
discuss the strategies I use to write unit tests I can trust.

## Technique: Scaffold the Project With Functional Tests

Although I just said that I don't like to overdo functional testing, we've
started this project with no unit tests and one functional test, and we're
about to add more. Early on in a project, the advantages of functional testing
outweigh its costs:

- Functional tests help us check that the production code portion of our
  walking skeleton is working.
- Functional tests don't pin down any of the *internal* interfaces of our code
  (e.g. classes and methods), so we're free to redesign those internals without
  changing the tests. This flexibility can be a boon early on in a project, when
  even the high-level architecture of the internals is still in flux.
- Functional tests run relatively quickly when there's not much code in the
  project. (On my machine, the "hello world" test takes 7 or 8 seconds to
  run—most of that being setup and teardown.)

That said, we can't rely on functional tests forever. As the project grows the
functional tests will take longer and longer to run, and it will become harder
and harder to write tests that *drive out* (i.e. force us to implement) the
specific behaviors we care about. We will eventually have to pivot our strategy
toward unit testing. Later on, we'll see how to make that transition smoothly
and safely.

For now, we'll forge ahead and write our first real functional test. The
question is, what behavior of our program do we want to test-drive first?

## Technique: Don't Go for the Gold

When people first start doing TDD, they often want to jump right into testing
the most fundamental, central behavior of their program. Since we're writing a
key-value store, the central behavior might be setting and getting several
different keys.

Patience is needed here. If you jump straight to the central behavior, you'll
lose much of the benefit of TDD. Bob Martin phrases this advice as "don't go
for the gold".

If we tried, right now, to implement setting and getting keys, we'd discover we
needed to implement a lot of other things. We'd have to parse the HTTP request.
We'd have to handle errors somehow (or just let them propagate) and make a
mental note to come back and test all those error cases. We'd have to decide
how to format responses. After perhaps half an hour or an hour of work, we'd
have one passing test, and a sinking feeling that not all of our code's
behavior is really tested. We'd also have a mental to-do list of error cases
and edge cases that we should really go back and test later. *This is exactly
the situation that TDD is intended to help us avoid.* TDD is a tool for
removing uncertainty and shortening our mental stack.

One technique for avoiding this situation is to start with the error cases,
edge cases, and degenerate cases. *We're going to avoid the central behavior of
the key-value store for as long as possible.* That way, when we do go to
implement it, it will be clean and simple: all the tricky bits will already be
handled.

With all of that in mind, the behavior I chose to test first was an
endpoint `/revisions/latest`, which returns the ID of the
current *revision* of the data.  You might notice that this
endpoint isn't described in the README! This is fine; the
README is, after all, just a sketch. I kind of expected
that, in the course of implementation, we'd discover
behavior that was needed but not documented in the README.

In choosing the `/revisions/latest` endpoint as the first
thing to test, I'm thinking ahead to another test I want
to write, again avoiding the central behavior: setting
a single key-value pair. I figure that if I have an endpoint
to get the current revision, I can assert that the revision
gets incremented by the set operation.

Here's the test. It's a straightforward modification of the
"hello world" test:

```ruby
require "net/http"
require "uri"
require "json"

describe "Fridge" do
  before :all do
    # ...
  end

  after :all do
    # ...
  end

  it "responds to a request for the latest revision" do
    expect(JSON.parse(Net::HTTP.get(URI("http://localhost:4567/revisions/latest"))))
      .to eq({"id" => 0})
  end
end
```

We're parsing the HTTP response body as JSON, and asserting it's equal to the hashmap
`{"id" => 0}`.

This test, of course, fails, because the server is returning `"Hello, World!"`
in response to every request. We can pass the test by changing the server code
to:

```ruby
get "*" do
  return 200, '{"id": 0}'
end
```

Some readers will no doubt find this troubling. We haven't implemented anything interesting! We're
just hardcoding exactly the response the test expects!

However, this stupid-simple approach to making the tests pass has benefits.
It's a technique that I call *calibrating the tests.*

## Technique: Calibrate Your Tests

One way to think about testing is that *tests are an
instrument* for detecting problems with our code. An
instrument, in science, is a device that measures something.
Thermometers are instruments, as are speedometers,
anemometers, and pretty much anything else that ends in
-ometer. To be useful, an instrument must be more legible
than the thing it is measuring. Instruments must also be
calibrated correctly, or their readings will be off.

A test is like an instrument: the fact that it passes does
not necessarily mean the code is correct. It only means that
the code and the test are in harmony—that they agree in some
way. We can thus only trust our tests to the degree that
they are simple, legible, and well-calibrated.

Tests, after all, are not infallible. Tests are just code,
and code can have bugs. Too often, I’ve encountered
tautological tests—tests that continue to pass even if you
delete the production code. A test that’s wrong or
tautological is worse than no test at all, since it gives us
false confidence that the code is working.

But how do we check that our tests are correct? We can’t
write tests for our tests! That would lead to an absurd
infinite regress, where we would need tests for our tests’
tests, and tests for those tests, on and on and on. We need
some way of bootstrapping our confidence in our tests. Or,
going back to the metaphor of “tests are an instrument”, we
need some way to calibrate the instrument before we use it.

So imagine you’re trying to calibrate a thermometer. You
have a glass tube full of mercury and you want to mark a
temperature scale on it. How would you do it?

Conveniently, there are two well-known temperatures you can
use as reference points: the boiling point and the freezing
point of water. So you introduce your thermometer to some
boiling water and make a mark for 100°C on the glass tube.
Then you dip it in ice water and mark 0°.

Note that when you do this, you’re not actually trying to
measure the temperature of the water. You already know the
temperature. You’re measuring the thermometer’s response.

You’re also testing both the thermometer and your
calibration process against your theory of how thermometers
work. If the level of the mercury didn’t change when you
moved the thermometer from hot to cold, or the mercury
appeared to expand rather than contract, you’d suspect
something was wrong with either the thermometer or your
calibration process.

This is why TDDers watch their tests fail before making them
pass. They’re testing the test against code that’s known to
be flawed, making sure that it fails when it’s supposed to.
When they first make a test pass, it’s usually in a way that
seems utterly trivial, like hardcoding the return value the
test expects. This, too, is part of the test’s calibration.

It may seem like overkill to always calibrate tests this
way, and in truth, I sometimes don’t bother. However, if you
don’t do it, you’ll occasionally run into issues that are
annoying to debug when you mistake them for a “real” test
failure:

The production code may not be loaded correctly by the test,
so the test always fails. The test may be referencing a
production class or function other than the one you’re
trying to test. This can happen because you misspelled the
class or function name, or because of a name collision
between two classes or functions.

Controlled studies of testing approaches suggest that
calibrating tests is not as useful as some TDDers claim.
These studies compare TDD to iterative test-last
development—that is, writing a small amount of code, and
then writing tests for that code. The studies’ results match
my subjective experience. The most important aspects of TDD
are that it helps you design testable interfaces and
supports refactoring, not that it helps you test your tests,
since tests ought to be very simple anyway. Bob Martin
posits that the thought processes involved in iterative
test-last development are identical to the thought processes
involved in TDD, which would explain why the studies found
no difference between them.

I do think that the concept of test calibration is
important, even if the practice is not, since it encourages
us to be humble about what our tests “know” or can achieve.

## Technique: Refactor Tests When They're Failing

At this point, our tests are passing, but that long line
with all the parentheses is pretty hard to read. I'd like
to refactor the test to make it clearer what's going on.

While the mantra of red-green-refactor urges us to only refactor
when our tests are passing, I think this is only advice
*when refactoring production code*. In this case, though,
we want to refactor the test code.

I prefer to refactor tests when they're failing. Seeing the
same failure message before and after the refactoring is a
richer source of information than a simple green bar, and
makes me more confident that I didn't accidentally change
the tests' behavior. If you refactor tests when they're
passing, it's all too easy to short-circuit assertions or do
the wrong setup. This can lead to tests that don't test the
scenario they're supposed to, or may even test nothing at
all.

So before we clean up our test, let's make it fail. We can
do that by changing the hardcoded JSON `{"id": 0}` in the
production code to `{"id": 1}`.

## Technique: Refactor to Fix Code Smells

Now we can refactor the test. When I refactor, I prefer to
first identify a specific *code smell* that I'm trying to get
rid of. A code smell is a structural problem with code that...

- is easy to spot
- often (though not always) hints at a deeper problem with
  the maintainability or correctness of the code.
- can be fixed by applying a particular sequence of
  refactorings.

The term _code smell_ found its way into the literature via
Martin Fowler and Kent Beck's book _Refactoring:
Improving the Design of Existing Code._ The book presents
a comprehensive set of refactorings and a less
comprehensive set of code smells.

I've attempted to build atop Fowler and Beck's work by
cataloguing the smells
I've found in my own code. You can find a list of these
in the reference chapter near the end of this book.

Why consider code smells at all?
Identifying code smells before you refactor makes the
process of refactoring less subjective. When you refactor,
it's all too easy to get lost in a maze of different design
approaches, or, at the opposite end of the spectrum,
doggedly pursue some "perfect" design that ends up being
impractical. I've found that teammates often differ in
their refactoring goals and this can lead to code "thrashing"
between designs as different maintainers pull it in
different directions.

When refactorings are based on code smells rather than
mere opinion or the pursuit of some grand goal, teammates
can have focused conversations about whether the code smell
is worth fixing and whether the refactoring fixes it. They
may still end up disagreeing, but at the very least they'll
understand each others' preferences better. Additionally,
I've found that refactoring incrementally, just fixing one
code smell at a time, often leads to elegant, simple designs
that I didn't foresee.

With all that in mind, let's consider: what code smell are we looking at here?
This is the line of code I want to change:

```ruby
expect(JSON.parse(Net::HTTP.get(URI("http://localhost:4567/revisions/latest"))))
  .to eq({"id" => 0})
```

All those nested method calls with their associated closing
parentheses are reminiscent of the "pyramid of doom", a
form of the _deep hierarchy_ smell. Deeply nested
syntactical structures are hard for humans to parse intuitively;
our linguistic hardware just isn't wired for it.

One refactoring to flatten out the syntactical hierarchy
is _Extract Variable_. Applying this refactoring
twice, we get:

```ruby
uri = URI "http://localhost:4567/revisions/latest"

response_body = Net::HTTP.get uri

expect(JSON.parse(response_body)).to eq({"id" => 0})
```

Note that there are other possible variables we could have
extracted, e.g. `expected = {"id" => 0}`. However, that
extraction wouldn't help fix the code smell.

The resulting code is much clearer. Now, our test assertion says exactly
what the test means: the response body should contain JSON
that parses to `{"id" => 0}`.

If we run the tests again, we get the expected failure,
letting us know that our refactoring is correct. We can
now go back and fix the production code so the tests pass.

<!-- At this point, there's another smell, a _data smear_. The
URI and HTTP request method (`GET`) are two halves of a
whole: both are used to form the HTTP request. However,
they're represented in different places and in different
ways: the URI is an object, while the `GET` is a mere
method on the `HTTP` object. To understand the whole HTTP
request as a unit, you thus have to visually scan and
mentally parse two different ideas. In fact, the very *idea*
that an HTTP request is a unit composed of method and URL
is obscured by this code.
-->

## Filling in the Details

Before we move on to testing the next endpoint, there are some
details we need to fill in. Our server is returning JSON
in the HTTP response body, but not setting the `Content-Type`
header, which might confuse some clients. Let's extend our
test to expose the problem:

```
it "responds to a request for the latest revision" do
  uri = URI "http://localhost:4567/revisions/latest"

  response = Net::HTTP.get_response uri

  expect(JSON.parse(response.body)).to eq({"id" => 0})
  expect(response["Content-Type"]).to eq "application/json"
end
```

One change I had to make to support this new assertion was
using the `Net::Http.get_response` method to make the  HTTP
request. This returns a response object which lets us access
the headers, whereas the `get` method just returned the
response body as a string.

This test fails in the expected way, so we can now fix
the production code:

```ruby
get "*" do
  return 200, {"Content-Type" => "application/json"}, '{"id": 0}'
end
```

## Technique: Analyze Behavioral Coverage With Mutation Testing

Now that I'm looking at this code more closely, I notice
that there's no test that makes an assertion about the
`200 OK` response code. Of course, just because there's no
assertion doesn't mean this behavior of the server is
untested. Perhaps `Net::HTTP.get_response` would raise an
exception if the request returned a non-successful response
code.

One technique we can use to find out if our tests cover this
behavior is to change the production code so it's obviously
wrong, and see if any tests fail. This technique is called
_mutation testing_.

I change the server code to this:

```ruby
get "*" do
  return 456, {"Content-Type" => "application/json"}, '{"id": 0}'
end
```

I run the tests and nothing fails. Looks like we need to
add an assertion.

```ruby
it "responds to a request for the latest revision" do
  uri = URI "http://localhost:4567/revisions/latest"

  response = Net::HTTP.get_response uri

  expect(JSON.parse(response.body)).to eq({"id" => 0})
  expect(response["Content-Type"]).to eq "application/json"
  expect(response.code).to eq 200
end
```

The test failure I get after this change is interesting:

```
Failure/Error: expect(response.code).to eq 200

  expected: 200
       got: "456"
```

Apparently, the `Net::HTTP` library represents the response
code as a string! Who knew?

I change the test assertion to account for this:

```ruby
expect(response.code).to eq "200"
```

Now the test fails in the expected way: just the value,
not the type, is wrong:

```
Failure/Error: expect(response.code).to eq 200

  expected: "200"
       got: "456"
```

I now go back to the production code and fix the mutation
I introduced, now confident that the tests will prevent me
from accidentally re-introducing it.

## Technique: Arrange, Act, Assert

It often improves test readability to organize each test into three sections:
setting up (_Arrange_), performing the action whose effects we want to
test (_Act_), and verifying that those effects happened (_Assert_).

Some people like to add comments at the beginning of each section, e.g.

```ruby
it "responds to a request for the latest revision" do
  # Arrange
  uri = URI "http://localhost:4567/revisions/latest"

  # Act
  response = Net::HTTP.get_response uri

  # Assert
  expect(JSON.parse(response.body)).to eq({"id" => 0})
  expect(response["Content-Type"]).to eq "application/json"
  expect(response.code).to eq 200
end
```

However, I find that simply putting blank lines between the sections creates
enough of a visual boundary to make the test easy to parse.

I only apply this technique when my tests don't fit on a single line. When
testing simple functions, it's usually overkill. For example, I'll write:

```ruby
describe "Pretty.format" do
  it "represents a short array as a one-liner" do
    expect(Pretty.format([1,2,3])).to eq "[1, 2, 3]"
  end
end
```

## Implementing "404 Not Found" Errors

All this thinking about HTTP statuses has reminded me that
our server should probably return a `404 Not Found` status
if the client requests a URL that doesn't exist. Currently,
the `get "*"` line in the server code will make the server
respond to *every* `GET` request as if it's asking for the
latest revision ID.

Here's a failing test for `404` responses:

```ruby
describe "Fridge" do
  # ...

  it "responds 404 to a request for a bogus URL" do
    response = http_client.request get "/blep"
    expect(response.code).to eq "404"
  end
end
```

To make this test pass, we just have to change `get "*"` to `get "/revisions/latest"`
in the production code. The `Sinatra` framework will generate the 404 response
for us when it gets a request for an endpoint that we haven't told it how to handle.

## Technique: Implement the Intent of the Tests

One way we could have made the `404` test pass is by adding this to the production code:

```ruby
get "/blep" do
  return 404, ""
end
```

Of course, that's not really what we want. The intent of our test is that *any*
URL other than `/revisions/latest` should return a 404.

We could continue down this path, writing another test like this:

```ruby
it "responds 404 to a request for a different bogus URL" do
  response = http_client.request get "/merp"
  expect(response.code).to eq "404"
end
```

And extending the production code like this:

```ruby
get "/blep" do
  return 404, ""
end

get "/merp" do
  return 404, ""
end
```

And at that point, we'd enter refactoring mode, contemplate the duplication, and
hopefully find a clean way to remove it.

While it's possible to test-drive like this to prove a point,
I find this way of working too plodding for everyday use. It also veers close to
the antipattern of _trying to prove correctness with tests_. How many bogus URLs
do our tests need to check to be sure that we've implemented `404` responses
correctly? The answer seems to be either "one" or "infinity".

I prefer to follow the intent of
the tests. I already know that I
want `404` to be the default case, so I let it be. I also think it's safe to assume
that anyone reading the code would start with a similar assumption.

You have to be careful of course, not to get carried away with this and implement
functionality or edge cases that aren't yet tested. Using Fridge as an example: we don't yet have
a test for the response body or `Content-Type` header of the `404` response, so I've
just let them default.

## Adding a `PUT` endpoint

We've now laid the groundwork for our first kind-of-real piece of functionality:
setting a key-value pair in the store and seeing the revision ID increment.

Here's a test for the endpoint:

```ruby
it "increments the latest revision number when you set a key" do
  http_client = Net::HTTP.new("localhost", 4567)

  http_client.request Net::HTTP::Put.new "/values/my-test-key"

  response = http_client.request Net::HTTP::Get.new "/revisions/latest"
  expect(JSON(response.body)).to eq({"id" => 1})
end
```

This fails with the expected error:

```
Failure/Error: expect(JSON(response.body)).to eq({"id" => 1})

  expected: {"id"=>1}
       got: {"id"=>0}
```

The simplest thing we can do to make the test pass is introduce a global
variable to store the current revision, and return it from the `get` endpoint.
Here's what the resulting code in `server.rb` looks like:

```ruby
require "sinatra"
require "json"

revision = 0

get "/revisions/latest" do
  return 200, {"Content-Type" => "application/json"}, JSON({"id" => revision})
end

put "*" do
  revision += 1
  return 204
end
```

At this point, alarm bells might be going off in your head. Global variables! Horrors!
Don't worry; soon we'll redesign this code so we don't have to use a global like this.
During that redesign, the test we just wrote will protect us from breaking the code's behavior.

But couldn't we have jumped straight to the better design? Theoretically, yes, but recall
that by getting the test to pass in the quickest, simplest way, we're calibrating the
test and double-checking that it's working correctly.

## Testing PUT Requests That Return 404 Responses

Now we have a problem: as we saw with the GET requests earlier, *any* `PUT` request will
increment the revision number, and that's
not exactly what we want. We should only increment the revision on a `PUT` request to
a URL like `/values/{some key}`.

Let's add a test that commits us to this intention.

```ruby
it "responds 404 to a PUT request for a bogus URL" do
  http_client = Net::HTTP.new("localhost", 4567)

  response = http_client.request Net::HTTP::Put.new "/mlem"
  
  expect(response.code).to eq "404"
end
```

When I ran this test for the first time, I was genuinely surprised by
the result. I expected this test to fail, and it did... but it also
caused *a different test* to fail!

After a short bout of head-scratching, I figured out why: here's the list of
tests in the order that they appeared in the file:

```ruby
it "responds 404 to a request for a bogus URL"
it "responds 404 to a request for a different bogus URL"
it "responds 404 to a PUT request for a bogus URL"
it "responds to a request for the latest revision"
it "increments the latest revision number when you set a key"
```

By default, RSpec will run the tests in the order in which they appear in the file.
With the production code in its current state, the third test increments the revision
number, which causes the fourth test (which asserts that the revision ID is 0) to
fail because the ID has been incremented to 1.

The test we just added has revealed the fact that our test suite suffers from
_test pollution_. Test pollution occurs when a test assumes that the previous
tests will leave the system in a particular state. In the presence of test pollution,
test suites become like a house of cards: any change to a
test is liable to cause other tests to fail.

At this point, I realized I shouldn't proceed without fixing the test pollution.
I stashed my change and

## Technique: Randomize Test Ordering to Expose Test Pollution



## Technique: Introduce a Naïve Unit-Testing Seam

## Technique: Beware the Framework

## Technique: Separate Unit and System Tests

## Technique: Remove Duplication With the Flocking Rules

## Technique: Refactor to Symmetry

## Technique: Keep Your Options Open With Contract Tests

## Technique: Implement Contracts Cheaply at First

## Technique: Always Represent Requirements Changes In The Contract

## Technique: Comb Your Lexicon For Tiny Objects

e.g. Revision

## Technique: Reproduce Bugs With Tests

## Technique: Design Up Front

## Technique: Write an Inefficient Reference Implementation

Implement a rolling hash with bigints and expensive modular
arithmetic—then try to write a more efficient implementation.

## Technique: Testing When You Don't Know the Right Answer

How to test that the rolling hash is "good"? I.e. that it
finds chunk boundaries at appropriate intervals.

## Technique: Avoid Realistic Values

TODO where does this fit?
