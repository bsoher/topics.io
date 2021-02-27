---
sort: 3
---

# A Potential Embedded Pitfall
This document describes a way to confuse Python and yourself when
embedding Python. It's only relevant if you have more than one Python
installed on the computer where your embedded code runs.

## Overview
If you have more than one Python installed and you invoke your embedded
Python app (and that includes launching the ICE Python DLL), it's possible to
use parts from two Pythons simultaneously at
runtime. Doing so definitely falls into the category of
'results undefined' and I recommend avoiding it.

## How Can This Happen?
To start with, recall that when Windows searches for a DLL, it searches a
number of locations before searching the PATH. That's an easy fact to
overlook and/or forget. Here's the official documentation:
https://msdn.microsoft.com/en-us/library/7d83bc18.aspx

When your executable
(.exe or .dll) that embeds Python asks Windows to load the Python runtime DLL
`python27.dll`, Windows loads the first `python27.dll` it finds on the DLL
search path. So far, so good.

Inside the ICE Python executable, we call `Py_SetPythonHome()` to tell Python where to find
its libraries (both the standard library and 3rd party libs like `numpy`).

If we point Python's home to one Python installation and Windows loads the runtime DLL
from a different installation, then the Python runtime will be mixing parts of two Python installs.
Presumably they would both be Python 2.7 and probably even binary compatible, in
which case things might just work fine. Maybe.

When mixing parts of two Pythons, you can get very confusing results when
querying Python about itself. For instance, `sys.version` might report
something like this --

```
Python 2.7.11 (v2.7.11:6d1b6a68f775, Dec  5 2015, 20:40:30) [MSC v.1500 64 bit (AMD64)] on win32
```

That's the signature of Python.org Python. However, `sys.exec_prefix`
might report this --

```
C:/Users/philip/Miniconda2
```

And `site.getsitepackages()` might report this --

```
C:/Users/philip/Miniconda2/lib/site-packages
```

The confusion comes from the fact that the `sys` module is written in C and
gets the version string from a hardcoded string in the runtime DLL. In this
case, that's being supplied by Python.org Python.

The `exec_prefix` and `site-packages` location are calculated from the value
we set when we called `Py_SetPythonHome()`, so they point to and are indeed
using a different Python.

## How To Avoid This Problem
Since Windows first searches for DLLs in the same directory as the
executable, that's the safest place to put your preferred Python runtime DLL to
ensure it will be found.

## How to Detect This Problem
Conflicting information from the `sys` module as in the example above is one way.

Another way is to use
[Process Explorer](https://technet.microsoft.com/en-us/sysinternals/processexplorer.aspx)
which can show you exactly which DLLs an excutable uses at runtime.
