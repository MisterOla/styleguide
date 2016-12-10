# Python

- [PEP8](#pep8)
- [General conventions](#general-conventions)
- [Python version support](#python-version-support)
    - [Python version 2 and version 3](#python-version-2-and-version-3)
    - [Default version in OS](#default-version-in-os)
- [Line wrapping](#line-wrapping)
- [Data structures](#data-structures)
- [Private interfaces](#private-interfaces)
- [Docstrings](#docstrings)
- [Logging](#logging)
- [pylint](#pylint)
- [flake8](#flake8)
- [Overall format](#overall-format)
- [References](#references)

## PEP8

[PEP8 - Style Guide for Python Code][PEP8] is THE guide for programming in python.  Pretty much everything there I adhere to.  The next sections mainly just clarifies certain parts of it.

## General conventions

- Indentation is 4 spaces, no tabs
- Target width is < 80 characters. Going beyond that is fine but be prepared for a lot of errors when running lint checkers.
- Remove all trailing spaces including on blank lines
- Use UTF-8 formatting without the BOM as that just seems to confuse editors
- Don't use `from modulename import *` unless it's by convention. For example fabric assumes you are going to import *.
- Use a `if __name__ = '__main__'` block which then calls a main function unless you are 100% sure this module will never be called directly. You can also turn off the top line from the script that's usually there to invoke the python interpreter if it's going to only be imported
- Prefer using modules in the standard library versus third party.  Primarily to have as few dependencies as possible.  As an example having a config file in yaml format would be really useful since I can store data structures and such, but it's not in the standard library.
    - Workaround for this is to use a `config.py` that's just a normal python file you include.
    - Prefer config.ini if there isn't a need for advanced data structures that ini files can't do without some hacking.
- 'Constants' (like [Private interfaces](#private-interfaces) this is by convention only) should be at top of file and all caps. The use of script wide constants are generally discouraged. If you have options that are used in several other functions then look in to making a class.

## Python version support

- Python 2.x - v2.6+ (needed because CentOS 6 has v2.6)
- Python 3.x - v3.3+ (added several more options to make v2 code run in v3)

### Python version 2 and version 3

Where possible code should be made to run in both the version 3 and version 2. At a minimum the following `__future__` imports should be used.

```python
from __future__ import (absolute_import, print_function, unicode_literals,
                        division)
```

Both `print_function` and `unicode_literals` were added in 2.6 thus this line would break 2.5 and older. If like me having to wrap the line to get it under 80 characters bothers you, you can remove `print_function` as that isn't required in 2.6. This is at the expense that it won't complain if you accidentally do a `print 'foo'` somewhere. Also division is only really needed when you expect a float returned on division. I still usually leave everything there in the hopes I'll eventually get used to it.

There are a couple good resources to help with writing code that's both compatible in 2 and 3. See [Porting to Python 3][Port2Python] and [python3porting.com][python3porting].

Will likely want to make use of [pyenv](https://github.com/yyuu/pyenv) and see my fork of [pyenv-installer](https://github.com/vrillusions/pyenv-installer).

### Default version in OS

Table of various operating systems and what versions of python they come with

| OS           | Python 2          | Python 3         |
| ------------ | ----------------- | ---------------- |
| CentOS 6.8   | 2.6.6             | n/a              |
| CentOS 7.2   | 2.7.5             | 3.4.5 (via EPEL) |
| Ubuntu 14.04 | 2.7.6             | 3.4.3            |
| Ubutnu 16.04 | 2.7.12 (optional) | 3.5.2            |

## Line wrapping

For continuing long lines I tend to use the second of these to:

```python
# Aligned with opening delimiter
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# More indentation included to distinguish this from the rest.
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)
```

The first way is more popular in other projects but if you're already calling a long function there's going to be a lot of wasted space.  I'd rather just double-indent and gain the horizontal space.  Note that the use of second form, called a hanging indent (even though it's actually the reverse or real meaning) varies from the first in that the first line should have nothing after the `(`. For example:

```python
# Bad
def long_function_name(var_one, var_two, var_three,
        var_four):
    print(var_one)

# Good
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)

# Also only double indent if needed. For example this is correct:
if somevar = some_really_long_function_name_here()
        and hello_world():
    print(
        'this is not a long sentence but at least '
        'the indentation is correct.')
    print('blah')

# This is not
def hello_world():
    print(
            'The double indent should not be here'
            'as it makes Guido cry')
    print('blah')
```

## Data structures

When a data structure spans multiple lines, format it as a hanging indent with only one item per line.

```python
# Good
print('blah')
some_words = (
        'word1',
        'word2',
        'word3',
    )
print(some_words)

# Bad - multiple items on a line and not hanging indent
print('blah')
some_words = (
    'word1','this should be on separate line',
    'word2',
    'word3',
)
print(some_words)
```

## Private interfaces

While python doesn't support a way to declare something is officially public or private. Instead a convention has come up. Anything prefixed with a single underscore should be considered private. Nothing is stopping you from importing it other than you shouldn't.  There's also the option of using two underscores.  This does some mangling of the actual name to avoid any collisions and I personally avoid the double underscore.

Prefer making objects private vs public. You can always make a private object public in the future but making a public object private could cause issues.

## Docstrings

The top file, all functions and all classes should have a docstring (also called a doc block).

Example doc block for top of file:

```python
"""Nifty Title.

This is a nifty module. Here I would put a longer description wrapped at 80
characters.

Requirements
    Python v2.6 or higher: Makes use of several functions introduced in 2.6

Environment Variables
    LOGLEVEL: overrides the level specified here. Choices are debug, info,
        warning, error, and critical. (default: 'warning')

"""
```

Example of a function doc block:

```python
def sample_def(meh=None):
    """This is a sample function.

    Long description would go here.

    Args:
        meh: I guess this would have something here

    Returns:
        The string 'hello'

    """
    return 'hello'
```

Example of a class doc block:

```python
class SampleClass():
    """This is a sample class.

    Just illustrating how a docblock would look.

    Attributes:
        blah: This stores some number of blahs

    """
    def __init__(self):
        """Inits SampleClass."""
        self._log = logging.getLogger(__name__)
        self.blah = 3

    def get_blah(self):
        """Returns the value of blah."""
        self._log.debug('Returning blah')
        return self.blah
```

These were all taken pretty much verbatim from [google's python style guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html#Comments).

## Logging

Logs MUST include a time stamp in ISO8601 format. Suggested format string:

```python
_LOGFORMAT = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
logging.basicConfig(level=logging.WARNING, format=_LOGFORMAT)
log = logging.getLogger(__name__)
```

Log levels should follow the same conventions laid out in the [python documentation](http://docs.python.org/2/howto/logging.html#logging-basic-tutorial) but also repeating here. These are in order from most verbose to least. So a log level of `ERROR` would only show `ERROR` and `CRITICAL` log messages:

- **DEBUG** - Information that is only of use to developers and possibly for troubleshooting issues. Should only be enabled during development.
- **INFO** - Information that may be of use to end users. An example of this is when contacting an external site. At `INFO` level just say "Now contacting (insert just the hostname here)" and then at `DEBUG` level it would change to "calling (full url with request parameters)".
- **WARNING** - For reporting on items that are not configured properly. Deprecation notices could go to this layer too. If whatever caused the message needs attention but didn't actually prevent anything (the python docs give an example of disk space starting to get low. This is the default log level.
- **ERROR** - When something did not work (e.g. invalid parameter to a function)
- **CRITICAL** - A serious error has happened that may cause the program to close. It's analogous to 'fatal' in other logging libraries but `CRITICAL` was chosen since in some situations like a web server where at `CRITICAL` level message may affect a specific connection but server is still running.

## Pylint

Use [pylint](http://www.pylint.org/) to help find common errors. The only thing that needs modified from a base install is to add `fh` as a valid variable name. To do this add `fh` to the end of `good-names` in your `.pylintrc` file. With default values this makes the final line `good-names=i,j,k,ex,Run,_,fh`.

## Flake8

Lately I've preferred [flake8][] over [pylint][]. Flake8 combines `pep8`, `pyflakes`, and McCabe complexity tests. There are additionally plugins available as well. Configuration for this is done by creating `tox.ini` with the following info:

```
[flake8]
# E123 requires opening and closing bracket (eg multi-line lists) to be at same
#   indentation. The opposite of this, E133, tests that it follows the same
#   rules as a hanging indent
# E226 tests for `1 + 1` instead of `1+1` (not in pep8 and I do the 'wrong' way
#   all the time)
# E265 requires any comment blocks to begin with '# ' (note the space).
#   Typically I don't add the space if temporarily commenting out code
ignore = E123,E226,E265
# This is the default
max-line-length = 79
# Unsure what default is but the recommended max is 10
max-complexity = 10
```

## Overall format

On executable files use `#!/usr/bin/env python` for python 2 and `#!/usr/bin/env python3` if something is specific to that version.  Note this may change in the future as python 3 becomes more popular.  Non executable files may omit this line.

Next line is `# -*- coding: utf-8 -*-`. Previously I would use the vi form of specifying text encoding, since that's the only editor I use.  The thing is this isn't for your editor which should already be set to utf-8.  This is for python to know that you have specifically said this file is utf-8.  Without this in python 2 the file will be read as ascii format.  In python 3 it defaults to utf-8.

A very minimal template follows:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""A one line description should go here.

Then a longer description would go here.
"""
from __future__ import (absolute_import, print_function, unicode_literals,
                        division)
import sys


def main(argv=None):
    """This is the main function."""
    if argv is None:
        argv = sys.argv
    print("Hello world")


if __name__ == "__main__":
    sys.exit(main())
```

## References

- [Style Guide for Python Code (PEP0008)][PEP8]
- [Porting to Python 3][Port2Python]
- [python3porting.com][python3porting] - This has more detail than other site
- [Code Like a Pythonista: Idiomatic Python](http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html) - Good into to the getchas in python
- [The Hitchhiker’s Guide to Python](http://docs.python-guide.org/en/latest/) - a current up to date guide on python
- [Best Practices for Python Scripting](http://cdn.oreillystatic.com/en/assets/1/event/27/Best%20practices%20for%20_scripting_%20with%20Python%203%20Paper.pdf) (pdf) - quick reference of sort
- [Python Project Howto](http://infinitemonkeycorps.net/docs/pph/) - how to structure a project for publishing
- [Google Python Style Guide](http://google-styleguide.googlecode.com/svn/trunk/pyguide.html) - One of the few style guides to be explicit in exactly how the doc string for modules and classes should be structured.


 [PEP8]: http://www.python.org/dev/peps/pep-0008/ "Style Guide for Python Code"
 [Port2Python]: http://docs.pythonsprints.com/python3_porting/index.html "Porting to Python 3"
 [python3porting]: http://python3porting.com/ "Python 3 Porting"
 [pylint]: http://www.pylint.org/ "Pylint"
 [flake8]: http://flake8.readthedocs.org/ "Flake8"
