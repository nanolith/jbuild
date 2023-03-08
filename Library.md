JBuild Library
==============

The low-level JBuild library is written in C. It is a simple object-relational
mapping of dependencies in a directed graph.


Dependency Graph
----------------

The graph is cached in a database, which makes it easier to load and store.
During the build process, the build target is looked up in the database. If the
target does not exist, an attempt is made to generate it. This lookup provides a
graph of dependencies. Each dependency is in turn looked up and placed on the
potential build stack. When all dependencies are loaded, these are reconciled
against files and directories on disk, peeling items off the stack as they are
verified as up to date or rebuilt. For verification, these are compared against
cached modification times and checksums, depending on configuration. If these
cached values do not match, then the resulting target is considered "stale" and
must be rebuilt. In turn, this means that all downstream targets that depend on
this target must also be rebuilt.

To keep track of failing builds or user interruption, each build is given a
unique number that is monotonically increased from the creation of the database.
This number is 64-bit, which gives quite a bit of headroom in the database. If
ever the number reaches `0xFFFFFFFFFFFFFFFF`, the entire database is rebuilt.
Every time a build starts, a new number is provided. This ensures that as
dependencies are updated, it's easy to keep track of which targets are stale
due to a depedency being newer than a target. This also ensures that the build
system tracks _changes_ and doesn't just assume that targets being newer than
dependencies means that the build is done. For instance, if the system clock or
a file modification date skews so that it appears that a particular target was
built in the _future_ even though the dependency has changed in the apparent
_past_, the change can still be detected because the cached modification time or
checksum is _different_. Thus, the new modification time or checksum is saved
along with the new build number, which ensures that any downstream targets are
correctly marked as stale. This idea is inspired by shake.

At the most basic level, a dependency is just a target that is depended upon by
another target. All targets have a unique key, a list of input dependencies, a
build plugin, and a list of outputs. Targets can be referenced in the database
by either their keys or their outputs. Targets that use outputs as keys are
alias targets that point to the real target as a dependency. A target can either
depend on a single output of another target or all outputs of another target.
The latter dependency style is important below.

A plugin consists of three functions: a refresh function, which determines
whether a target is stale and if so, provides an updated target, a build
function, which given a target will build this target, and a third function. The
third function will be described momentarily. This is a quite basic interface,
but it provides a few interesting utilities. For instance, we can write a target
that takes a list of wildcards as input, and expands these into a sorted list of
filenames as output. A target that depends on this target (all outputs
dependency) would automatically be marked as stale if this list changed.  The
plugin can determine whether this list has changed by comparing the stored
checksum of the previous list with the checksum of this list. Thus, we can have
a build target that converts a wildcard string of `.c` files into a list of
files. A second build target can depend on this list, which produces a list of
`.o` files. A third build target can depend on this list of `.o` files in order
to build an executable or library. A fourth build target can take as input any
`.c` file matching a pattern and produce a `.o` file.

To facilitate pattern matching, this third function is used. Given a `NULL`
string as input, this function will return true if it supports pattern matching
and false otherwise. All pattern matching targets are stored in a special table
so that they can be referenced looking up dependency rules and no explicit
dependency can be found.  When a pattern match is requested, the string of the
requested dependency is passed to the pattern match function. If this function
returns true, then this target can be used to build this dependency. If this
function returns false, then this target can't be used to build this dependency.
If multiple pattern matching targets match a particular dependency, then this is
an error.

Database
--------
