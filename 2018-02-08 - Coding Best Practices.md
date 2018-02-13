# Brainstorming topics

* Travis CI
    * Rule of thumb: keeps additional things "in sync" with your GitHub state.
* vim or emacs
* bash on Windows
* coding best practices—commenting, structure, naming
* software architecture—writing a new plugin, what to keep in mind to avoid future hackery

Curtis wishes someone knew numpy / scipy and gave an awesome workshop on it.

* Slated for late March!

# Coding best practices

## Working state to working state

When you make a change, you want to know that it didn't break anything. But you can only know that if the previous version was working. So naturally, we want to have a working version as soon as possible:

1. Get to some working state. MINIMUM. New project, just create a trivial little program.
2. Then, it's possible to make incremental changes.
3. Each of these incremental changes is a Git commit. This lets you use e.g. `git bisect`

So each commit needs to be TESTABLE.

Test-driven development—TDD. Worth reading about!

1. Write a unit test that does what you want syntactically. It won't compile / run.
    ```python
    import myRandomModule as random
    for i in range(10):
        r = random.random()
        assert r >= 0.0 and r < 1.0
    ```
    Code will not run. There is no `myRandomModule` module.
2. Get the code to compile and run. But failing. In this case, we create that module, and make that method `random()`.
3. Then, make the test pass. All we need here would be `return 0`. NOW we're in business, you have working state.
4. You can then iterate: either make the test more thorough (i.e. assert more than just [0, 1) here). Or you can make the algorithm better.

I used a unix command called `watch` to repeatedly execute tests. Then you can edit your sources and watch the test results MAGICALLY change.

Corollary: tests should run FAST.

In general, we have the Edit-Compile-Run-Debug cycle. It goes by [various names like that](https://www.google.com/search?q=edit+compile+run+debug+cycle).
If you have things like GUIs, you can look at "robot" frameworks. You can look at mock frameworks in some cases (for things like faking a database connection). But it sucks.
Corollary: write small, testable functions whenever possible. SEPARATION OF CONCERNS.

Rules of thumb to achieve this:

1. A method should only do one thing.
2. A class should only do one thing.

E.g. if a method does four major things, write:
```python
def bigFunction():
     subFunction1()
     subFunction2()
     subFunction3()
     subFunction4()

def subFunction1():
     ...
```
A great book about unit testing—and getting to "islands of good code"—is Working Effectively with Legacy Code by Michael Feathers. I have a copy of this book if anyone wants to borrow it.

## Class architecture tangent

Tangent: favor composition over inheritance. A `Person` _has_ `Arm`s. They ARE NOT arms.

WRONG:
```java
public class Person extends Head, Torso, Arm, Leg {
}
```

RIGHT:
```java
public class Person {
  Head head;
  Torso torso;
  Arm leftArm, rightArm;
  Leg leftLeg, rightLeg;
  ...
}
```
Planning is good. BUT, for large systems, it will be developed iteratively. SMALL iterations as above.

## Code style - WHEN IN ROME - CONSISTENCY

It's helpful to mimic the style of whatever codebase you're in.

For the ImageJ project, we document the desired style [here](http://imagej.net/Coding_style).

Good code is self-documenting. The method name describes what it does. The only time you really want a comment is when the code itself doesn't read like human language. One reason for this might be if it violates the [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment).
```python
x = sin(x) # NB: Be warned! This takes the square root.
```
Above, the comment is important. Supposing we cannot change the function name for some reason (e.g. it comes from an external library), we document what it does RIGHT THERE.

Corollary: minimize chance of [action at a distance errors](https://blog.codefx.org/java/java-10-var-type-inference/#Avoiding-8220Action-At-A-Distance8221-Errors)—i.e., changing code in some place should not lead to a seemingly unrelated error far away.

The other time I use comments is to preface a BLOCK of code. But in that case, strongly consider making a sub-method instead.

```java
public void go() {
    doStuff();
    
    // Perform some steps to compute a working variable.
    int x = initialValue();
    x += theSum();
    
    doMoreStuff(x);
}
```

I favor judicious use of whitespace in the vicinity of line comments, to block off what the comment is talking about. Above, we know that the comment is talking about the lines that work with x, and _not_ really the `doMoreStuff(x)` call or any later lines.

## Versioning

Pragmatically, most of the world uses [SemVer](http://semver.org/) now. This means the version number tells you whether the change is backwards compatible, and whether the change added new API, or just fixed bugs. All of SciJava and ImageJ is done with SemVer. This is in contrast to "marketing" versioning, where going from v5 to v6 historically meant "it is shiny and new with lots of new features."

In Java land, we use Maven. This differentiates between SNAPSHOT and RELEASE versions. The rule of thumb is: always depend on release versions, for reproducibility. See [Versioning](http://imagej.net/Versioning) and [Reproducibility](http://imagej.net/Architecture#Reproducible_builds).

The good news is: with Git, you always have versions: it's the commit hashes. But best practice is to use tags (a feature of git) to bookmark your releases so that people can always find them easily.

### When should you make a new version, versus calling it by a new name?

"It depends." But rule of thumb is: do you need the old thing to stick around so that old downstream code using it still works? If yes, but you also want to provide the new thing at the same time, then you really have two things. An example of this is [Apache Commons Math](http://commons.apache.org/proper/commons-math/). This is Java library for doing math. With v3, they chose a new package prefix `org.apache.commons.math3`. The old v2 is `org.apache.commons.math`. With that scheme, it is possible for to mix and match usage of both in the same environment easily.

Ask yourself: what's the situation with backwards compatibility?

Michael: in LaTeX, there was a function for making tables. It was communicated that this function would break, and people shouldn't use it if they wanted things to keep working. Assumptions/paradigm or mechanistically, the behavior changed. Not a BUG per se. Just better behavior.

## Licensing

You should have one. :-)

## How much should you plan?

Classic debate between ["agile"](https://en.wikipedia.org/wiki/Agile_software_development) vs. the more traditional whitepaper-driven (e.g. ["waterfall"](https://en.wikipedia.org/wiki/Waterfall_model)) design paradigms.

Either way, first gather some [requirements](https://en.wikipedia.org/wiki/Software_requirements). This is very important. Understand what users need.

But it is easy to get too detailed. Flowcharting every class can be temporarily helpful when the design is complex... but that documentation will never last (in my experience). You get "documentation skew"—the docs are wrong, after the code has evolved.

This tradeoff is (in some ways) a variation of the classic "I can do it right, or I can do it fast." We want some balance.

Very general corollary: It's important to make your process as smooth and easy as possible.

# Next meeting

Next meeting will be SOFTWARE ARCHITECTURE. Curtis will talk about buzzwords like _extensibility_, _encapsulation_, _dependency injection (DI)_, _interoperability_ vs. _integration_, _application containers_ and _plugins_.
