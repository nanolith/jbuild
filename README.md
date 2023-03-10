jbuild
======

This is Justin's Opinionated Build System. Currently, it's just a set of notes
that I'm making for the sort of build system I want, but I'm going to write a
build library and eventually an implementation.

I _really_ like Shake. But, it's been practically impossible to drive adoption
for this tool. Haskell has been a hard sell, and it's even a harder sell if just
the build system is a Haskell DSL. That being said, Shake is probably where this
project will start.

My goals are to build up a reasonable library for defining dependencies, rules,
and actions; from there, serializing dependency data into and out of a database;
I want to port this C library to Rust, Python, Ruby, Haskell, Java, and .NET; I
also want to build a simple build language and compiler that can transform build
rules, build actions, and plugin callbacks into executable code. Incremental
builds should occur nearly as fast as it takes to load a binary, execute it, and
scan the filesystem against dependencies saved in the database.

As a stretch goal, VM languages like Java and .NET should have a daemon service
that can amortize the cost of booting these VMs so that plugins written in
higher level languages have a lower performance penalty. But, this is
significantly far in the future. For now, I want to document the things I want,
experiment in writing a minimal library in model checked C, and then set up a
basic language and compiler.
