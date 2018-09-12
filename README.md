
# java.nio

This package does not yet contain any implementation.  Just an idea.

## Summary

There should be a standard, simple, idiomatically Clojure way to access the file
system.

The old `java.io.File` class is bad and should be avoided when possible.

## History

### clojure.java.io

Clojure provides a standard library with polymorphic I/O utility functions,
[clojure.java.io](https://clojure.github.io/clojure/clojure.java.io-api.html).

That library began life as a `clojure.contrib` package, and was "promoted"
to being part of the Clojure project in May of 2010, for Clojure 1.2.

### Java History

At the time `clojure.java.io` became a thing, Java 6 was quite mature (released
December 2006) and Java 7 was still over a year away.

Java contains some much-reviled classes for common pieces of functionality,
including `java.util.Date` and `java.io.File`.  As of Java 6, there was no
alternative to using `java.io.File` (without resorting to JNI).

### Clojure's Java History

It seems that the first few versions of Clojure did not specifically document
a required version of Java, but since Clojure uses java.util.concurrent, it
seems reasonable to assume that Java 5 was required.

With the reducers library added in Clojure 1.5 (March 2013), Java 6 was
required.  Java 6 became a requirement upon the release of Clojure 1.6
in March of 2014 --the same month Java 8 was released.

Numerous Clojure applications were built on top of Java 6.  When there were
new Clojure versions, it was desired that Clojure applications should be
able to upgrade their version of Clojure, without necessarily having to
upgrade their JVM.  Java 6 remained the requirement for Clojure 1.6 - 1.8.
This meant, of course, that the Clojure language could not use any features
that were not present in Java 6.

As of the release of Clojure 1.9, the requirement remains Java 1.6; however,
Java 1.8+ is "recommended".

The latest development release as of this writing, 1.10.0-alpha7, requires
Java 1.8+.

### Support for Optional Versions

Of course, it *is* possible to write code in a way that runs on Java 6, but
takes advantage of features of Java 8, when present.  The code can do a runtime
check, and invoke one set of functionality or the other, depending on its
environment.

In fact, the `fs` project (see below) did just that.  Initially, my idea was to
develop a "unified" file I/O, that would use, and would be based on, `java.nio`
as much as possible, by providing implementations of `nio` functionality using
`java.io.File`.

But if Clojure 1.10 is going to require Java 8 anyway, any Clojure 1.10 library
should be able to feel free to assume that `java.nio` is present.


## Motivation

### Improvements of NIO

The `java.nio` package is better.

There are no Clojure-specific reasons why it's better, so to understand why it's
better, one can simply read documents about why it's better in Java programs.

I would not inline that information here, but I could perhaps find some existing
documents and link to them.  (TODO).  Until then, understanding why `java.nio` is
better than `java.io` is left as an exercise for the reader...

### Having Standard Libraries

Again, this has nothing in particular to do with Clojure, nor with file I/O.  In
general, when a group of programmers writing in a given language need to access
the same functionality, there are many advantages to having them use the same
library, rather than everyone inventing their own interfaces.

As above, I could find and link to some essays about why standard libraries are
so important, but for now, I'll take it as a given that they are.

### Challenges of Newer Javas

It has been correctly pointed out that Clojure never imposed a maximum version
of Java.  If a Clojure programmer wanted to use functionality that was new in
Java 8, they could certainly do so.  They could require Java 8 for their
project.  The fact that the Clojure language did not require it does not prevent
a specific, non-language library or application from doing so.

In addition to all the usual reasons why it's easier and better to use
a standard library rather than rolling your own, newer versions of Java
introduced an additional complication: variadic arguments.

For example, `java.nio.file.Paths` contains a static method,
```public static Path get(String first, String... more)```

This has one required String argument, and any number of additional String
arguments.  It could, for instance, be called with either of the following:
```Path newPath = Paths.get("/usr", "bin", "java");```
or
```Path newPath = Paths.get("/usr/bin", "java");```

In the JVM, the "real" method signature is `Path get(String, String[])`.
The Java compiler packages the extra arguments into a `String[]`, and
passes that array to the method.  It has to be a `String[]`.  It can't
be a generic array of Objects, let alone any sort of Collection.

This can be done from Clojure, of course, but it's a little low-level.

From Java, the variadic method can also be called with zero additional
arguments:
```Path newPath = Paths.get(filepath);```

Since the JVM supports polymorphism via argument lists, it *could*
define a method that simply takes a String.  But (at least in this case),
it doesn't.  The above code also is translated by the complier into an
invocation that passes a `String[]`, albeit an empty one.

In order to call these methods from Clojure, you need to create String
arrays.  Again, hardly the end of the world once you understand how to
do it (and why it's necessary), but it's definitely ugly, and definitely
a stumbling block for people who aren't highly experienced with Clojure.

In searching GitHub to see to what extent all this has already been done, I
found another person addressing the same question:
[Why wrap java.nio?](https://github.com/ToBeReplaced/nio.file/blob/master/WHY.md)

### JIRA

Clojure support for `java.nio.file` has been requested:
[Support java.nio.file.Path in clojure.java.io](https://dev.clojure.org/jira/browse/CLJ-2333)
