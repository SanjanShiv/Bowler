---
id: basics-usage
title: Usage
---

Bowler provides two primary mechanisms for refactoring.  It provides a
[fluent API](fixers) for standalone applications to import and use, and it also
provides a simple command line tool for executing one or more refactoring scripts
on a given set of directories or files.


## Command Line

Bowler has multiple commands, which allow you to debug or execute code refactoring
scripts in the most convenient format for you.  See the
[Command Reference](api-commands.md) for more details.

```bash
bowler <command> [<options> ...]
```

## Fluent API

Bowler uses a "fluent" `Query` API to build refactoring scripts through a series
of selectors, filters, and modifiers.  Many simple modifications are already possible
using the existing API, but you can also provide custom selectors, filters, and
modifiers as needed to build more complex or custom refactorings.  See the
[Query Reference](api-query.md) for more details.

Using the query API to rename a single function, and generate an interactive diff from
the results, would look something like this:

```python
query = (
    Query(<paths to modify>)
    .select_function("old_name")
    .rename("new_name")
    .diff(interactive=True)
)
```

## Refactoring Scripts

Refactoring scripts combine the mechanisms above to provide self-contained, reusable
components for refactoring large code bases.  They consist of one or more queries or
fixers in a single Python file, ready to be imported and executed by Bowler.  This
allows the user to run these predefined or parameterized refactors on future code,
enabling long term benefits from the initial effort.

For example, if we have a simple refactoring script named `rename_func.py`:

```python
from bowler import Query


old_name, new_name = sys.argv[1:]
(
    Query(".")
    .select_function(old_name)
    .rename(new_name)
    .idiff()
)
```

That script can then be executed directly as a normal python application, or with the
following Bowler command, to interactively rename and function called `foo` to `bar`,
including all references:

```bash
bowler run rename_func.py -- foo bar
```

## Testing lib2to3 Patterns

When working with Bowler, understanding and testing `lib2to3` patterns can be challenging for new users. To make this process easier, you can use the following utility function to experiment with patterns in a REPL environment:

```python
from pprint import pprint

from fissix import pytree, pygram
from fissix.patcomp import PatternCompiler
from fissix.pgen2 import driver

d = driver.Driver(pygram.python_grammar.copy(), pytree.convert)
pc = PatternCompiler()

def test(pattern, code):
    pat = pc.compile_pattern(pattern)
    for node in d.parse_string(code).pre_order():
        for match in pat.generate_matches([node]):
            pprint(match)

# Example usage:
pattern="""
call=power<
  any*
  trailer<"." "foo">
  trailer<"(" args=any* ")">
>
"""
# Prints out details about both the f.foo and bar.foo calls
test(pattern,'a = f.foo(123) + bar.foo("adf")\n')
```

This function allows you to quickly iterate and test patterns, making it easier to write and understand them. You can copy this function into your own scripts or REPL sessions to experiment with different patterns.
