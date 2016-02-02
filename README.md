# Jupyter Scala

Jupyter Scala is a Scala kernel for [Jupyter / IPython](http://ipython.org/).
It aims at being a modular and up-to-date alternative to other Scala kernels or
notebook UIs.

[![Build Status](https://travis-ci.org/alexarchambault/jupyter-scala.svg?branch=master)](https://travis-ci.org/alexarchambault/jupyter-scala)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/alexarchambault/jupyter-scala?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=body_badge)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.alexarchambault.jupyter/scala_2.11.7.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.alexarchambault.jupyter/scala_2.11.7)

## Table of contents

...

## Quick start

First ensure you have [IPython / Jupyter](http://ipython.org/) installed.
Running `ipython --version` should print a value >= 3.0. See [#ipython-installation]
if it's not the case.

Then download and run the Jupyter Scala launcher with
```
$ curl -L -o jupyter-scala https://git.io/vzhRi && chmod +x jupyter-scala && ./jupyter-scala && rm -f jupyter-scala
```

This downloads the bootstrap launcher of Jupyter Scala, then runs it. If no previous version of it is already installed,
this simply sets up the kernel in `~/.ipython/kernels/scala211`. Note that on first launch, it will download its
dependencies from Maven repositories. These can be found in `~/.jupyter-scala/bootstrap`.
Once installed, the downloaded launcher can be removed, as it copies itself in `~/.ipython/kernels/scala211`.

For the Scala 2.10.x version, you can do instead
```
$ curl -L -o jupyter-scala-2.10 https://git.io/vzhR7 && chmod +x jupyter-scala-2.10 && ./jupyter-scala-2.10 && rm -f jupyter-scala-2.10
```

The Scala 2.10.x version shares its bootstrap dependencies directory, `~/.jupyter-scala/bootstrap`, with the
Scala 2.11.x version. It installs itself in `~/.ipython/kernels/scala210`.

You can check that a kernel is installed with
```
$ ipython kernelspec list
```
which should print one line per installed IPython / Jupyter kernel.

## Why

There are already a few notebook UIs or Jupyter kernels for Scala out there
([IScala](https://github.com/mattpap/IScala),
[ISpark](https://github.com/tribbloid/ISpark),
[scala-notebook](https://github.com/Bridgewater/scala-notebook),
[spark-notebook](https://github.com/andypetrella/spark-notebook),
[Toree](https://github.com/apache/incubator-toree) - formerly spark-kernel).
Most of them share very little code with other projects, and target only a few
narrow uses.
Jupyter Scala aims at being a lot more modular than them, which allows for much more sharing of things
developed for particular uses of notebooks with other uses. Pretty-printing, completion,
helpers for plotting can be re-used in a variety of contexts (fast prototyping, data science, day-to-day
graphical REPL, etc.).
Jupyter Scala also aims at being closer to what
[Ammonite](https://github.com/lihaoyi/Ammonite) achieves in the terminal in terms of completion or
pretty-printing. Also, like with Ammonite, users interact with the interpreter via a Scala API
rather than ad-hoc hard-to-reuse-or-automate special commands. Jupyter Scala also publishes
this API in a separate module (`scala-api`), which allows to write external libraries
that interact with the interpreter. In particular it has a Spark bridge, that straightforwardly
adds Spark support to a session. More bridges like this should come soon, to interact with
other big data frameworks, or for plotting. 

Thanks to this modularity, Jupyter Scala shares its interpreter and most of its API with
[ammonium](https://github.com/alexarchambault/ammonium), the fork of Ammonite it is based on.
One can switch from notebook-based UIs to the terminal more confidently, with all that
can be done on one side, being possible on the other (a bit like Jupyter / IPython allows with its
`console` and `notebook` commands, but with the additional niceties of Ammonite, also available in ammonium,
like syntax highlighting of the input). In teams where some people prefer terminal interfaces
to web-based ones, some people can use Jupyter Scala, and others its terminal cousins, according to their tastes.


## Special commands / API

The content of an instance of a [`jupyter.api.API`](https://github.com/alexarchambault/jupyter-scala/blob/master/api/src/main/scala/jupyter/api/API.scala)
is automatically imported when a session is opened (a bit like Ammonite does with its "bridge").
Its methods can be called straightaway, and replace so called "special" or "magic" commands in other notebooks
or REPLs.

### show

```scala
show(value)
```

Print `value` - its [pprint](https://github.com/lihaoyi/upickle-pprint/) representation - with
no truncation. (Same as in Ammonite.)

### classpath.add

```scala
classpath.add("organization" % "name" % "version")
```

Add Maven / Ivy module `"organization" % "name" % "version"` - and its transitive dependencies - to the classpath.
Like in SBT, replace the first `%` with two percent signs, `%%`, for Scala specific modules. This
adds a Scala version suffix to the name that follows. E.g. in Scala 2.11.x, `"organization" %% "name" % "version"`
is equivalent to `"organization" % "name_2.11" % "version"`.

Can be called with several modules at once, like
```scala
classpath.add(
  "organization" % "name" % "version",
  "other" %% "name" % "version"
)
```

(Replaces `load.ivy` in Ammonite.)

### classpath.addInConfig

```scala
classpath.addInConfig("config")(
  "organization" % "name" % "version",
  "organization" %% "name" % "version"
)
```

Add Maven / Ivy modules to a specific configuration. Like in SBT, which itself follows what Ivy does,
dependencies are added to so called configurations. Jupyter Scala uses 3 configurations,

* `compile`: default configuration, the one of the class loader that compiles and runs things,
* `macro`: configuration of the class loader that runs macros - it inherits `compile`, and initially has `scala-compiler` in it, that some macros require,
* `plugin`: configuration for compiler plugins - put compiler plugin modules in it.

### classpath.dependencies

```scala
classpath.dependencies: Map[String, Set[(String, String, String)]]
```

Return the previously added dependencies (values) in each configuration (keys).

### classpath.addRepository

```scala
classpath.addRepository("repository-address")
```

Add a Maven / Ivy repository, for dependencies lookup. For Maven repositories, add the base address
of the repository, like
```scala
classpath.addRepository("https://oss.sonatype.org/content/repositories/snapshots")
```
For Ivy repositories, add the base address along with the pattern of the repository, like
```scala
classpath.addRepository(
  "https://repo.typesafe.com/typesafe/ivy-releases/" +
    "[organisation]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)" +
    "[revision]/[type]s/[artifact](-[classifier]).[ext]"
)
```

By default, `~/.ivy2/local`, Central (`https://repo1.maven.org/maven2/`), and
Sonatype releases (`https://oss.sonatype.org/content/repositories/releases`) are used.
Sonatype snapshots (`https://oss.sonatype.org/content/repositories/snapshots`) is also
added in snapshot versions of Jupyter Scala.

### classpath.repositories

```scala
classpath.repositories: Seq[String]
```

Returns a list of the previously added repositories.

### classpath.addPath

...

### classpath.classLoader (advanced)

```scala
classpath.classLoader(config: String): ClassLoader
```

Returns the `ClassLoader` of configuration `config` (see above for the available configurations).

### classpath.addedClasses (advanced)

```scala
classpath.addedClasses(config: String): Map[String, Array[Byte]]
```

Returns a map of the byte code of classes generated during the REPL session. Each line or cell
in a session gets compiled, and added to this map - before getting loaded by a `ClassLoader`
and run.

The `ClassLoader` used to compile and run code mainly contains initial and added
dependencies, and things from this map.

### classpath.configs (advanced)

...

### classpath.onPathsAdded (advanced)

...

### classpath.path (advanced)

...

## Writing libraries using the interpreter API

Most available classes (`classpath`, `eval`, `setup`, `interpreter`, ...) from a notebook
are:

* defined in the `scala-api` module (`"com.github.alexarchambault.jupyter" % "scala-api_2.11.7" % "0.3.0-M2"`) - or its dependencies,
* available implicitly from a notebook (`implicitly[ammonite.api.Classpath]` would be the same as just `classpath`).

This allows to write libraries that can easily interact with Jupyter Scala.

## IPython installation

Check that you have [IPython](http://ipython.org/) installed by running
`ipython --version`. It should print a value >= 3.0. If it's
not the case, a quick way of setting it up consists
in installing the [Anaconda](http://continuum.io/downloads) Python
distribution, and then running

    $ pip install --upgrade "ipython[all]"

`ipython --version` should then print a value >= 3.0.

## Examples

Some example notebooks can be found in the [examples](https://github.com/alexarchambault/jupyter-scala/tree/master/examples)
directory: you can follow [macrology 201](https://github.com/alexarchambault/jupyter-scala/blob/master/examples/tutorials/Macrology.ipynb) in a notebook,
use compiler plugins like [simulacrum](https://github.com/alexarchambault/jupyter-scala/blob/master/examples/libraries/Simulacrum.ipynb) from notebooks,
use a type level library to [parse CSV](https://github.com/alexarchambault/jupyter-scala/blob/master/examples/libraries/PureCSV.ipynb),
setup a notebook for [psp-std](https://github.com/alexarchambault/jupyter-scala/blob/master/examples/libraries/psp-std.ipynb)
etc. **More to come.**


## Internals

Jupyter Scala uses the Scala interpreter of [ammonium](https://github.com/alexarchambault/ammonium),
in particular its `interpreter` and `interpreter-api` modules. The interaction with Jupyter
(the Jupyter protocol, ZMQ concerns, etc.) are handled in a separate project,
[jupyter-kernel](https://github.com/alexarchambault/jupyter-kernel). In a way, Jupyter Scala is just
a bridge between these two projects!

The API as seen from a Jupyter Scala session is defined
in the `scala-api` module, that itself depends on the `interpreter-api` module of ammonium.
The core of the kernel is in the `scala` module, in particular with an implementation
of an `Interpreter` for jupyter-kernel, based on `interpreter` from ammonium,
and implementations of the interfaces / traits defined in `scala-api`.
It also has a third module, `scala-cli`, which deals with command-line argument parsing,
and launches the kernel itself. The launcher consists in this third module.

The launcher itself is generated with [coursier](https://github.com/alexarchambault/coursier)...
class loader isolation...

## Compiling it

...

Released under the Apache 2.0 license, see LICENSE for more details.
