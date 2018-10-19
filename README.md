# Megaparsec

[![License FreeBSD](https://img.shields.io/badge/license-FreeBSD-brightgreen.svg)](http://opensource.org/licenses/BSD-2-Clause)
[![Hackage](https://img.shields.io/hackage/v/megaparsec.svg?style=flat)](https://hackage.haskell.org/package/megaparsec)
[![Stackage Nightly](http://stackage.org/package/megaparsec/badge/nightly)](http://stackage.org/nightly/package/megaparsec)
[![Stackage LTS](http://stackage.org/package/megaparsec/badge/lts)](http://stackage.org/lts/package/megaparsec)
[![Build Status](https://travis-ci.org/mrkkrp/megaparsec.svg?branch=master)](https://travis-ci.org/mrkkrp/megaparsec)

* [Features](#features)
    * [Core features](#core-features)
    * [Error messages](#error-messages)
    * [Alex support](#alex-support)
    * [Character and binary parsing](#character-and-binary-parsing)
    * [Lexer](#lexer)
* [Documentation](#documentation)
* [Tutorials](#tutorials)
* [Performance](#performance)
* [Comparison with other solutions](#comparison-with-other-solutions)
    * [Megaparsec vs Attoparsec](#megaparsec-vs-attoparsec)
    * [Megaparsec vs Parsec](#megaparsec-vs-parsec)
    * [Megaparsec vs Trifecta](#megaparsec-vs-trifecta)
    * [Megaparsec vs Earley](#megaparsec-vs-earley)
* [Related packages](#related-packages)
* [Prominent projects that use Megaparsec](#prominent-projects-that-use-megaparsec)
* [Links to announcements and blog posts](#links-to-announcements-and-blog-posts)
* [Authors](#authors)
* [Contribution](#contribution)
* [License](#license)

This is an industrial-strength monadic parser combinator library. Megaparsec
is a feature-rich package that strikes a nice balance between speed,
flexibility, and quality of parse errors.

## Features

The project provides flexible solutions to satisfy common parsing needs. The
section describes them shortly. If you're looking for comprehensive
documentation, see the [section about documentation](#documentation).

### Core features

The package is built around `MonadParsec`, an MTL-style monad transformer.
All tools and features work with all instances of `MonadParsec`. You can
achieve various effects combining monad transformers, i.e. building a
monadic stack. Since the common monad transformers like `WriterT`, `StateT`,
`ReaderT` and others are instances of the `MonadParsec` type class, you can
wrap `ParsecT` *in* these monads, achieving, for example, backtracking
state.

On the other hand `ParsecT` is an instance of many type classes as well. The
most useful ones are `Monad`, `Applicative`, `Alternative`, and
`MonadParsec`.

Megaparsec includes all functionality that is available in Parsec plus
features some combinators that are missing in other parsing libraries:

* `failure` allows to fail reporting a parse error with unexpected and
  expected items.
* `fancyFailure` allows to fail reporting custom error messages.
* `withRecovery` allows to recover from parse errors “on-the-fly” and
  continue parsing. Once parsing is finished, several parse errors may be
  reported or ignored altogether.
* `observing` allows to “observe” parse errors without ending parsing (they
  are returned in `Left`, while normal results are wrapped in `Right`).

In addition to that, Megaparsec features high-performance combinators
similar to those found in Attoparsec:

* `tokens` makes it easy to parse several tokens in a row (`string` and
  `string'` are built on top of this primitive). This is about 100 times
  faster than matching a string token by token. `tokens` returns “chunk” of
  original input, meaning that if you parse `Text`, it'll return `Text`
  without any repacking.
* `takeWhile` and `takeWhile1` are about 150 times faster than approaches
  involving `many`, `manyTill` and other similar combinators.
* `takeP` allows to grab n tokens from the stream and returns them as a
  “chunk” of the stream.

Megaparsec is about as fast as Attoparsec if you write your parser carefully
(see also [the section about performance](#performance)).

Megaparsec can currently work with the following types of input stream
out-of-the-box:

* `String` = `[Char]`
* `ByteString` (strict and lazy)
* `Text` (strict and lazy)

It's also simple to make it work with custom token streams, and Megaparsec
users have done so many times.

### Error messages

Megaparsec has well-typed error messages and the ability to use custom data
types to adjust the library to specific domain of interest. No need to use a
shapeless bunch of strings.

Megaparsec 7 introduced the `ParseErrorBundle` data type that helps to
manage multi-error messages and pretty-print them easily and efficiently.
That version of the library also made the practice of displaying offending
line the default.

### Alex support

Megaparsec works well with streams of tokens produced by tools like Alex.
The design of the `Stream` type class has been changed significantly in
versions 6 and 7, but user can still work with custom streams of tokens
without problems.

### Character and binary parsing

Megaparsec has decent support for Unicode-aware character parsing. Functions
for character parsing live in the
[`Text.Megaparsec.Char`](https://hackage.haskell.org/package/megaparsec/docs/Text-Megaparsec-Char.html)
module. Similarly, there is
[`Text.Megaparsec.Byte`](https://hackage.haskell.org/package/megaparsec/docs/Text-Megaparsec-Byte.html)
module for parsing streams of bytes.

### Lexer

[`Text.Megaparsec.Char.Lexer`](https://hackage.haskell.org/package/megaparsec/docs/Text-Megaparsec-Char-Lexer.html)
is a module that should help you write your lexer. If you have used `Parsec`
in the past, this module “fixes” its particularly inflexible
`Text.Parsec.Token`.

`Text.Megaparsec.Char.Lexer` is intended to be imported using a qualified
import, it's not included in `Text.Megaparsec`. The module doesn't impose
how you should write your parser, but certain approaches may be more elegant
than others. An especially important theme is parsing of white space,
comments, and indentation.

The design of the module allows you quickly solve simple tasks and doesn't
get in your way when you want to implement something less standard.

Since Megaparsec 5, all tools for indentation-sensitive parsing are
available in `Text.Megaparsec.Char.Lexer` module—no third party packages
required.

`Text.Megaparsec.Byte.Lexer` is also available for users who wish to parse
binary data.

## Documentation

Megaparsec is well-documented. See the [current version of Megaparsec
documentation on Hackage](https://hackage.haskell.org/package/megaparsec).

## Tutorials

You can find Megaparsec tutorials
[here](https://markkarpov.com/learn-haskell.html#megaparsec-tutorials). They
should provide sufficient guidance to help you to start with your parsing
tasks. The site also has instructions and tips for Parsec users who decide
to migrate to Megaparsec.

## Performance

Despite being flexible, Megaparsec is also fast. Here is how Megaparsec
7.0.0 compares to Attoparsec 0.13.2.2 (the fastest widely used parsing
library in the Haskell ecosystem):

Test case         | Execution time | Allocated | Max residency
------------------|---------------:|----------:|-------------:
CSV (Attoparsec)  |       76.50 μs |   397,784 |        10,544
CSV (Megaparsec)  |       64.69 μs |   352,408 |         9,104
Log (Attoparsec)  |       302.8 μs | 1,150,032 |        10,912
Log (Megaparsec)  |       337.8 μs | 1,246,496 |        10,912
JSON (Attoparsec) |       18.20 μs |   128,368 |         9,032
JSON (Megaparsec) |       25.45 μs |   203,824 |         9,176

The benchmarks were created to guide development of Megaparsec 6 and can be
found [here](https://github.com/mrkkrp/parsers-bench).

If you think your Megaparsec parser is not efficient enough, take a look at
[these
instructions](https://markkarpov.com/megaparsec/writing-a-fast-parser.html).

## Comparison with other solutions

There are quite a few libraries that can be used for parsing in Haskell,
let's compare Megaparsec with some of them.

### Megaparsec vs Attoparsec

[Attoparsec](https://github.com/bos/attoparsec) is another prominent Haskell
library for parsing. Although the both libraries deal with parsing, it's
usually easy to decide which you will need in particular project:

* *Attoparsec* is sometimes faster but not that feature-rich. It should be
  used when you want to process large amounts of data where performance
  matters more than quality of error messages.

* *Megaparsec* is good for parsing of source code or other human-readable
  texts. It has better error messages and it's implemented as monad
  transformer.

So, if you work with something human-readable where size of input data is
usually not huge, just go with Megaparsec, otherwise Attoparsec may be a
better choice.

Since version 6, Megaparsec features the same fast primitives that
Attoparsec has, so in many cases the difference in speed is not that big.
Megaparsec now aims to be “one size fits all” ultimate solution to parsing,
so it can be used even to parse low-level binary formats.

### Megaparsec vs Parsec

Since Megaparsec is a fork of Parsec, we are bound to list the main
differences between the two libraries:

* Better error messages. We test our error messages using numerous
  QuickCheck (generative) tests. Good error messages are just as important
  for us as correct return values of our parsers. Megaparsec will be
  especially useful if you write a compiler or an interpreter for some
  language.

* Megaparsec 6 can show the line on which parse error happened as part of
  parse error. This makes it a lot easier to figure out where the error
  happened.

* Some quirks and “buggy features” (as well as plain bugs) of original
  Parsec are fixed. There is no undocumented surprising stuff in Megaparsec.

* Better support for Unicode parsing in `Text.Megaparsec.Char`.

* Megaparsec has more powerful combinators and can parse languages where
  indentation matters out-of-the-box.

* Comprehensive test suite covering nearly 100% of our code. Compare that to
  absence

* We have benchmarks to detect performance regressions.

* Better documentation, with 100% of functions covered, without typos and
  obsolete information, with working examples. Megaparsec's documentation is
  well-structured and doesn't contain things useless to end users.

* Megaparsec's code is clearer and doesn't contain “magic” found in original
  Parsec.

* Megaparsec has well-typed error messages and custom error messages.

* Megaparsec can recover from parse errors “on the fly” and continue
  parsing.

* Megaparsec allows to conditionally process parse errors *inside your
  parser* before parsing is finished. In particular, it's possible to define
  regions in which parse errors, should they happen, will get a “context
  tag”, e.g. we could build a context stack like “in function definition
  foo”, “in expression x”, etc. This is not possible with Parsec.

* Megaparsec is faster and supports efficient operations on top of `tokens`,
  `takeWhileP`, `takeWhile1P`, `takeP` like Attoparsec.

If you want to see a detailed change log, `CHANGELOG.md` may be helpful.
Also see [this original
announcement](https://mail.haskell.org/pipermail/haskell-cafe/2015-September/121530.html)
for another comparison.

### Megaparsec vs Trifecta

[Trifecta](https://hackage.haskell.org/package/trifecta) is another Haskell
library featuring good error messages. It's probably good, but also
under-documented, and has unfixed [bugs and
flaws](https://github.com/ekmett/trifecta/issues). Other reasons one may
question choice of Trifecta is his/her parsing library:

* Complicated, doesn't have any tutorials available, and documentation
  doesn't help at all.

* Trifecta can parse `String` and `ByteString` natively, but not `Text`.

* Trifecta's error messages may be different with their own features, but
  certainly not as flexible as Megaparsec's error messages in the latest
  versions.

* Depends on `lens`. This means you'll pull in half of Hackage as transitive
  dependencies. Also if you're not into `lens` and would like to keep your
  code “vanilla”, you may not like the API.

[Idris](https://www.idris-lang.org/) has recently switched from Trifecta to
Megaparsec which allowed it to [have better error messages and fewer
dependencies](https://twitter.com/edwinbrady/status/950084043282010117?s=09).

### Megaparsec vs Earley

[Earley](https://hackage.haskell.org/package/Earley) is a newer library that
allows to safely (it your code compiles, then it probably works) parse
context-free grammars (CFG). Megaparsec is a lower-level library compared to
Earley, but there are still enough reasons to choose it over Earley:

* Megaparsec is faster.

* Your grammar may be not context-free or you may want introduce some sort
  of state to the parsing process. Almost all non-trivial parsers require
  something of this sort. Even if your grammar is context-free, state may
  allow to add some additional niceties. Earley does not support that.

* Megaparsec's error messages are more flexible allowing to include
  arbitrary data in them, return multiple error messages, mark regions that
  affect any error that happens in those regions, etc.

* The approach Earley uses differs from the conventional monadic parsing. If
  you work not alone, people you work with, especially beginners, will be
  much more productive with libraries taking more traditional path to
  parsing like Megaparsec.

IOW, Megaparsec is less safe but also more powerful.

## Related packages

The following packages are designed to be used with Megaparsec (open a PR if
you want to add something to the list):

* [`hspec-megaparsec`](https://hackage.haskell.org/package/hspec-megaparsec)—utilities
  for testing Megaparsec parsers with with
  [Hspec](https://hackage.haskell.org/package/hspec).
* [`cassava-megaparsec`](https://hackage.haskell.org/package/cassava-megaparsec)—Megaparsec
  parser of CSV files that plays nicely with
  [Cassava](https://hackage.haskell.org/package/cassava).
* [`tagsoup-megaparsec`](https://hackage.haskell.org/package/tagsoup-megaparsec)—a
  library for easily using
  [TagSoup](https://hackage.haskell.org/package/tagsoup) as a token type in
  Megaparsec.

## Prominent projects that use Megaparsec

The following are some prominent projects that use Megaparsec:

* [Idris](https://github.com/idris-lang/Idris-dev)—a general-purpose
  functional programming language with dependent types
* [Hledger](https://github.com/simonmichael/hledger)—an accounting tool
* [MMark](https://github.com/mmark-md/mmark)—strict markdown processor for
  writers
* [Stache](https://github.com/stackbuilders/stache)—Mustache templates for
  Haskell
* [Language Puppet](https://github.com/bartavelle/language-puppet)—library
  for manipulating Puppet manifests

## Links to announcements and blog posts

Here are some blog posts mainly announcing new features of the project and
describing what sort of things are now possible:

* [Megaparsec 7](https://markkarpov.com/post/megaparsec-7.html)
* [Evolution of error messages](https://markkarpov.com/post/evolution-of-error-messages.html)
* [A major upgrade to Megaparsec: more speed, more power](https://markkarpov.com/post/megaparsec-more-speed-more-power.html)
* [Latest additions to Megaparsec](https://markkarpov.com/post/latest-additions-to-megaparsec.html)
* [Announcing Megaparsec 5](https://markkarpov.com/post/announcing-megaparsec-5.html)
* [Megaparsec 4 and 5](https://markkarpov.com/post/megaparsec-4-and-5.html)
* [The original Megaparsec 4.0.0 announcement](https://mail.haskell.org/pipermail/haskell-cafe/2015-September/121530.html)

## Authors

The project was started and is currently maintained by Mark Karpov. You can
find the complete list of contributors in the `AUTHORS.md` file in the
official repository of the project. Thanks to all the people who propose
features and ideas, although they are not in `AUTHORS.md`, without them
Megaparsec would not be that good.

## Contribution

Issues (bugs, feature requests or otherwise feedback) may be reported in
[the GitHub issue tracker for this project](https://github.com/mrkkrp/megaparsec/issues).

Pull requests are also welcome (and yes, they will get attention and will be
merged quickly if they are good).

## License

Copyright © 2015–2018 Megaparsec contributors\
Copyright © 2007 Paolo Martini\
Copyright © 1999–2000 Daan Leijen

Distributed under FreeBSD license.
