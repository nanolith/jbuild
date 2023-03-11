jbuild Language
===============

The scripting language for jbuild is purposefully simple. Anything more than
describing dependencies or explicit tool execution steps should be done in a
plugin. The language also has to be trivially and obviously extensible in order
to ensure that we can pass parameters to a plugin in the most natural way.

The first requirement is easily accomplished by borrowing syntax, or at least
semantics, from `make`.  However, unlike `make`, we don't use a tab delimited
mechanism for denoting build steps. Likewise, we skip a lot of the odd variables
and patterns. These things belong in plugins. Instead, we focus on denoting the
relationship between target and inputs.  As for the second requirement, we need
a way to denote the type of an argument so we can perform type checking at
runtime for plugins. However, we don't want to use a "line noise" oriented
mechanism such as is found in Perl. Instead, we can focus on a limited sequence
of types that can be checked at runtime, as well as some rules we can carry
forward at compile time to keep types consistent. First, we support symbols,
double quoted strings, integers, decimals with a decimal point, single quoted
character constants, lists of a single type denoted by square brackets, and
complex object types denoted by curly braces, which may be a sequence of zero or
more symbols followed by values. This type system is similar to JSON, but just
slightly more terse. Unlike JSON, the symbol type ensures that we can produce an
object sequence that is a bit more natural to read. Compare this JSON sequence
to our object sequence.

JSON:

```json
    { "link_options" : {
        "linker" : "GNU LD",
        "max_size" : 47000,
        "ram_size" : 10240
    } }
```

jbuild:

```
    link_options {
        linker "GNU LD"
        max_size 47000
        ram_size 10240
    }
```
