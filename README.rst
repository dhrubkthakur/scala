################################################################################
                              THE SCALA REPOSITORY
################################################################################

This document describes the Scala core (core library and compiler) repository
and how to build it. For information about Scala as a language, you can visit
the web site http://www.scala-lang.org/

Part I. The repository layout
--------------------------------------------------------------------------------                            

Follows the file layout of the Scala repository. Files marked with a † are not
part of the repository but are either automatically generated by the
build script or user-created if needed.  This is not a complete listing. :: 
  scala/
   +--build/                    Build products output directory for ant.
   +--build.xml                 The main Ant build script.
   +--dist/                     The destination folder for Scala distributions.
   +--docs/                     Documentation and sample code.
   +--lib/                      Pre-compiled libraries for the build.
   |   +--fjbg.jar              The Java byte-code generation library.
   |   +--scala-compiler.jar    The stable reference ('starr') compiler jar
   |   +--scala-library.jar     The stable reference ('starr') library jar
   |   +--scala-library-src.jar A snapshot of the source used to build starr.
   |   ---ant/                  Support libraries for ant.
   +--pull-binary-libs.sh       Pulls binary artifacts from remote repository.
   +--push-binary-libs.sh       Pushes new binary artifacts and creates sha.
   +--README.rst                The file you are currently reading.
   +--src/                      All the source files of Scala.
   |   +--actors/               The sources of the Actor library.
   |   +--compiler/             The sources of the Scala compiler.
   |   +--library/              The sources of the core Scala library.
   |   ---swing/                The sources of the Swing library.
   +--target/       †           Build products output directory for sbt.
   +--test/                     The Scala test suite.
   ---tools/                    Developer utilities.



Part II. Building Scala with SABBUS
--------------------------------------------------------------------------------

SABBUS is the name of the Ant build script used to compile Scala. It is mostly
automated and takes care of managing the dependencies.

^^^^^^^^^^^^^^^^^^^^^^^^
        LAYERS:
^^^^^^^^^^^^^^^^^^^^^^^^
In order to guarantee the bootstrapping of the Scala compiler, SABBUS builds
Scala in layers. Each layer is a complete compiled Scala compiler and library.
A superior layer is always compiled by the layer just below it. Here is a short
description of the four layers that SABBUS uses, from bottom to top:

- ``starr``: the stable reference Scala release which is shared by all the
  developers. It is found in the repository as 'lib/scala-compiler.jar' and
  'lib/scala-library.jar'. Any committable source code must be compiled directly
  by starr to guarantee the bootstrapping of the compiler.

- ``locker``: the local reference which is compiled by starr and is the work
  compiler in a typical development cycle. When it has been built once, it is
  “frozen” in this state. Updating it to fit the current source code must be
  explicitly requested (see below).

- ``quick``: the layer which is incrementally built when testing changes in the
  compiler or library. This is considered an actual new version when locker is
  up-to-date in relation to the source code.

- ``strap``: a test layer used to check stability of the build.

^^^^^^^^^^^^^^^^^^^^^^^^
  DEPENDANT CHANGES:
^^^^^^^^^^^^^^^^^^^^^^^^
SABBUS compiles, for each layer, the Scala library first and the compiler next.
That means that any changes in the library can immediately be used in the
compiler without an intermediate build. On the other hand, if building the
library requires changes in the compiler, a new locker must be built if
bootstrapping is still possible, or a new starr if it is not.


^^^^^^^^^^^^^^^^^^^^^^^^
REQUIREMENTS FOR SABBUS:
^^^^^^^^^^^^^^^^^^^^^^^^
The Scala build system is based on Apache Ant. Most required pre-compiled
libraries are part of the repository (in 'lib/'). The following however is
assumed to be installed on the build machine:

- A Java runtime environment (JRE) or SDK 1.6 or above.
- Apache Ant version 1.7.0 or above.
- bash (via cygwin for windows)
- curl


Part III. Common use-cases
--------------------------------------------------------------------------------
- ``./pull-binary-libs.sh``

  Downloads all binary artifacts associated with this commit.  This requires
  internet access to http://typesafe.artifactoryonline.com/typesafe.

- ``ant -p``

  Prints out information about the commonly used ant targets. The interested
  developer can find the rest in the XML files.

- ``ant`` or ``ant build``

  A quick compilation (to quick) of your changes using the locker compiler.

  - This will rebuild all quick if locker changed.
  - This will also rebuild locker if starr changed.

- ``ln -s build/quick/bin qbin`` (once):
- ``ant && qbin/scalac -d sandbox sandbox/test.scala && qbin/scala -cp sandbox Test``
  
  Incrementally builds quick, and then uses it to compile and run the file
  ``sandbox/test.scala``. This is a typical debug cycle.

- ``ant replacelocker``
  
  "unfreezes" locker by updating it to match the current source code.

  - This will delete quick so as not to mix classes compiled with different
    versions of locker.

- ``ant test``

  Tests that your code is working and fit to be committed.

  - Runs the test suite and bootstrapping test on quick.
  - You can run the suite only (skipping strap) with 'ant test.suite'.

- ``ant docs``
  Generates the HTML documentation for the library from the sources using the
  scaladoc tool in quick.  Note: on most machines this requires more heap than
  is allocate by default.  You can adjust the parameters with ANT_OPTS.
  Example command line::
    ANT_OPTS="-Xms512M -Xmx2048M -Xss1M -XX:MaxPermSize=128M" ant docs

- ``ant dist``
  
  Builds a distribution.

  - Rebuilds locker from scratch (to make sure it bootstraps).
  - Builds everything twice more and compares bit-to-bit the two builds (to
    make sure it is stable).
  - Runs the test suite (and refuses to build a distribution if it fails).
  - Creates a local distribution in 'dists/latest'.

- ``ant clean``

  Removes all temporary build files (locker is preserved).

- ``ant locker.clean``

  Removes all build files.

- ``ant all.clean``

  Removes all build files (including locker) and all distributions.

Many of these targets offer a variant which runs with -optimise enabled.
Optimized targets include build-opt, test-opt, dist-opt, fastdist-opt,
replacestarr-opt, replacelocker-opt, and distpack-opt.

Part IV. Contributing to Scala
--------------------------------------------------------------------------------

If you wish to contribute, you can find all of the necessary information on
the official Scala website: www.scala-lang.org.

Specifically, you can subscribe to the Scala mailing lists, read all of the
available documentation, and browse the live github repository.  You can contact
the Scala team by sending us a message on one of the mailing lists, or by using
the available contact form.

In detail:

- Scala website (links to everything else):
  http://www.scala-lang.org

- Scala documentation:
  http://docs.scala-lang.org

- Scala mailing lists:
  http://www.scala-lang.org/node/199

- Scala bug and issue tracker:
  https://issues.scala-lang.org

- Scala live git source tree:
  http://github.com/scala/scala

If you are interested in contributing code, we ask you to sign the
[Scala Contributor License Agreement](https://www.lightbend.com/contribute/cla/scala),
which allows us to ensure that all code submitted to the project is
unencumbered by copyrights or patents.

Before submitting a pull-request, please make sure you have followed the guidelines
outlined in our `Pull Request Policy <https://github.com/scala/scala/wiki/Pull-Request-Policy>`_.

------------------



Thank you!

The Scala Team
