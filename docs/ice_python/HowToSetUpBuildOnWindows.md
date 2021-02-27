---
sort: 1
---

# Windows Build Environ Setup

This document describes how to set up a build environment for ICE Python on Windows.

## Install a Compiler

I use [Microsoft Visual C++ Compiler for Python 2.7](https://www.microsoft.com/en-us/download/details.aspx?id=44266).

## Set Paths at Boot Time

Most (all?) versions of MSVC come with a batch file called `VCVARSALL.BAT` that adds the compiler's
directories to the PATH and INCLUDE path. It's expected that you'll
call  `VCVARSALL.BAT` from `autoexec.bat` or whatever passes for Windows initialization these days.

Personally, I run Powershell and I've converted `VCVARSALL.BAT` to a Powershell script. It's in SVN.

