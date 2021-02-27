---
sort: 2
---

# Embed Python Under Win/Linux
This document describes the general technique of how to embed Python into a
C/C++ application under Windows and Linux.

The examples given are for a C++ program that uses Anaconda/Minconda Python 2.7.
These specifics affect a few paths and compliler flags, but the basic concepts
remain the same regardless of source language (C or C++) and Python version.

## Introduction
Embedding Python allows an executable to run a Python interpreter within its
process space and to exchange data with the Python code being executed.

It isn't difficult, but there's some nuance to be aware of.

## Linking
To start with, when linking one's C++ executable to a library, one can link
statically or dynamically. It's not a requirement for a Python distribution to
provide a static library, and few of them do so. It's best to ignore static
linking unless you have a compelling need.

Linking is fairly different under Windows and Linux, so I discuss them
separately.

## Windows Linking
Under Windows, during the link step it's sufficient to include the fully-qualified path to Python's
import library file (e.g. `C:/Users/philip/Miniconda2/libs/python27.lib`).

## Linux Linking
Under Linux, the Makefile uses the `python-config` utility which tells us
which compile and link flags to use.

We also supply an extra link flag. The `-L` flag takes a directory as an
argument (e.g. `-L/home/me/miniconda2/lib`) and tells the linker to search
that directory first when searching for libraries to link. (For details,
see the `ld` man page.)

The reason we pass this is because on Linuxes that use Python 2.7 as the
system Python, there will be a `libpython2.7.so` (and also possibly a
static library `libpython2.7.a`) in the linker's search path by default.
This might be the
same Python _version_ as the Python you'll be using at runtime, but it's
probably from a different distribution/provider.

Is it safe to link against one library and then use a different one at
runtime? Maybe. I'd rather not. Adding an explicit library search path
to the link step with `-L$(PYTHON_LIBPATH)` ensures we link against the same
library we'll use at runtime.

## How to Provide the Python Runtime Library
Since the executable is dynamically linked to Python, we have to ensure
Python's runtime library (`python27.dll/libpython2.7.so`) is available at runtime.
Not only that, we want to ensure that the runtime library of our preferred
Python is loaded, not just the runtime library of whatever Python happens
to be first in the library search path.

The techniques for doing so differ under Windows and Linux, so I'll treat
them separately.

## Windows: Where to Put the Python Runtime DLL
The best place for `python27.dll` to reside is in the same directory as your
executable file. That's the first place Windows will look for DLLs that your executable
attempts to load. Windows' DLL search path order is documented here:
https://msdn.microsoft.com/en-us/library/7d83bc18.aspx

If you don't put the Python runtime DLL in the same directory as your
executable, you can encounter the problem described here: PotentialEmbeddedPitfall.

## Linux: Where to Put the Python Runtime DLL
When one's control over the environment is limited (as it is in this case,
where the scanner probably won't let us write to locations like `/usr/lib`),
we can use a feature of Linux executables called the "rpath" (where the "r"
stand for runtime library, I think).

The rpath is defined at compile time in the Makefile, and
it allows us to put the runtime libraries anywhere we want. There are some
useful properties of rpaths worth highlighting here --
 * One can specify multiple rpaths in an executable
 * rpaths are searched in the order specified
 * If an rpath points to a directory that doesn't exist, Linux doesn't care
 * rpaths can contain the special entry `$ORIGIN` which refers to the
 same directory as the executable, and one can reference subdirectories from
 that (e.g. `$ORIGIN/my_project/libs`).

There's instructions specific to this project below, but if you want a good
general reference on the subject, here's one:
http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html

The utility `ldd` tells you which libraries an executable loads
(e.g. `ldd my_executable`).

### rpath Syntax
The syntax required to use `$ORIGIN` in a Makefile is tricky because (a)
it's a linker option, not a compiler option, and we have to ask the compiler
to pass it to the linker, and (b) `$` is a metacharacter in Makefile syntax.
Here's an example of working syntax --
```
g++ -Wl,-rpath,'$$ORIGIN' -o my_executable my_source.cpp
```

The `-Wl,` tells GCC to pass what follows the comma to the linker. The double `$$`
is a Makefile escape sequence that collapses to `$` in the linker command.
You can add to the path inside the single quotes, like so --
```
g++ -Wl,-rpath,'$$ORIGIN/my_project/libs' -o my_executable my_source.cpp
```


## Where to Put Everything Else
Most of Python's standard library is implemented in `.py` files, and they're not
contained in `python27.dll`.

The `.py` files of the standard library have to be provided at runtime -- along
with `site-packages` -- as a subdirectory of the directory containing the executable.

## A Sample Directory Structure
Here's an explanation of each file in the directory listing below --
 * `simple.exe` is our C/C++ executable.
 * `python27.dll` is copied from Miniconda. (There's another copy in
 `./Miniconda2/python27.dll`, but Windows won't look there for it.)
 * Miniconda2 is a copy of Miniconda, including the standard library, numpy and
 all the other goodies.
 * `do_stuff.py` is our Python sample code.

```
Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         3/13/2016   4:52 PM            Miniconda2
-----         1/19/2016   1:08 PM    3021824 python27.dll
-----         3/13/2016   5:03 PM        785 do_stuff.py
-----         3/13/2016   4:56 PM     179712 simple.exe
```


## Setting Python Home in the C++ Code
It's useful or perhaps necessary to give Python a hint about where to find
the standard library by calling `Py_SetPythonHome()`. Using the directory
structure above, the call would look something like this --

```
Py_SetPythonHome("C:/path/to/your/executable/Miniconda2");
```



## Side Note: Libraries Types Under Windows
Under *nix, static libraries generally end in ".a" and dynamic
libraries in ".so" (also ".dyld" under OS X). Under Windows, static libraries
have the suffix ".lib" and dynamic have the suffix ".dll".

However, Windows has
a 3rd kind of library called an import library. Import libraries provide hints to the
linker when linking one's code against a DLL. Import libraries share the
".lib" suffix with static libraries which makes them difficult to tell apart.
One way to do so is with this command, which produces little
to no output for an import library and lots of output for a static library:
```
lib.exe /LIST:your_mystery_file.lib
```

`lib.exe` is provided by MSVC in its bin directory alongside `cl.exe` and `dumpbin.exe`.

In the case of Python, `python27.lib` is an import library rather than a
static library.

## Side Notes: Why We Can't Use Zipped Libraries
It's possible to zip the standard library and provide it as
as single file (thanks to 
[PEP 273](https://www.python.org/dev/peps/pep-0273/) which provides for importing
modules from .zip files). However, PEP 273 specifically states that only
pure Python files may be imported:

   Any files may be present in the zip archive, but only files
   *.py and *.py[co] are available for import.  Zip import of
   dynamic modules (*.pyd, *.so) is disallowed.

As a result, standard library modules written in C (like sqlite3) can't be
imported from a zipped standard library. Naturally, this also applies to
3rd party libraries written in C, like numpy.

For our application, this rather limits the usefulness of the feature provided
by PEP 273.