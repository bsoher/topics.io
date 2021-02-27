---
sort: 2
---

# How to Release/Upload New Vespa Wheel

## Overview
This explains how to build `pip`-installable wheels for Vespa.

### How To Build the Wheel

1. Start a command prompt and change to directory that contains `setup.py`.
1. Remove any build caches from previous wheel building --

```
 python setup.py clean --all
```

1. Build the wheel --

```
 python setup.py bdist_wheel --python-tag py27
```

That's it! Your wheel will be in the `dist` directory.

### About `setup.py`

Prior to version 0.9.0. Vespa's `setup.py` had a number of features and
responsibilities. As of 0.9.0, it's much smaller and does very little.

In fact, `setup.py` isn't even distributed with Vespa. Users are instructed to
install Vespa from a wheel file via `pip` (which basically just unzips the
Vespa wheel), so `setup.py` couldn't influence
the installation process even if we wanted it to.

It's possible to do a local (developer) install with `python setup.py install`,
although offhand I can't think of a reason for doing so. The only reason
you need `setup.py` is to build wheels.

### `MANIFEST.IN` and `setuptools`

`MANIFEST.IN` defines which files get included in the wheel. See [Python's doc on MANIFEST.IN](https://docs.python.org/2/distutils/sourcedist.html#specifying-the-files-to-distribute).

Vespa's `setup.py` uses `setuptools`, and we pass the `include_package_data` to `setuptools.setup()` in `setup.py`. That tells it to read `MANIFEST.IN` to know which files to include in the wheel.

### Dueling Vespas

If you're modifying `setup.py`, chances are that you're a Vespa developer and you have a `vespa.pth` file in your `site-packages` directory. When you test your Vespa wheel, you'll then have a `vespa` directory in `site-packages` in addition to `vespa.pth`. When Python looks in site-packages to resolve an `import vespa.something` statement, it will have two Vespas from which to choose. I don't know what Python's behavior is in this case and I suspect it's undefined.

The practical upshot is that you need to remove `site-packages/vespa.pth` while you're working on `setup.py` and testing wheels.


## How to Release a New Vespa Version

### How To Upload Vespa Wheels

This document describes how to upload Vespa wheels to PyPI.

*Note. Once you load a version to PyPI, the same version can NOT be reloaded. So be careful.*

Just FYI - One workaround is to upload 'release candidate' versions, e.g. if you upload 1.2.15rc1 initially you can still upload 1.2.15 later.  

### Introduction

As of this writing, uploading wheels requires some help from 3rd party packages. See the "One-time Setup Steps" section below if you have not already set these packages up, yet.

If you find these steps to be too much trouble, remember that you can always upload the wheel files to scion instead. That would create a small inconvenience for the user.

If the Vespa wheels are on PyPI, the user can type `pip install vespa-suite` and pip will find and install the right version of Vespa for the user's system.

### Credit

Some of the information in this document comes from here: https://hynek.me/articles/sharing-your-labor-of-love-pypi-quick-and-dirty/

### Assumptions

This document assumes the following --
 * You have already built the Vespa wheel
 * You have an account on the test PyPI
 * You have an account on the production (real) PyPI
 * You have followed the one-time setup steps in this document (see below)

### Uploading Wheels

*Upload to the test PyPI if you want -- *

```
twine upload --repository testpypi dist/*
```

You can find this upload at [https://test.pypi.org/project/Vespa-Suite/] 

To use this test version to upgrade your version of Vespa, type:

```
pip install --index-url https://test.pypi.org/simple vespa-suite --upgrade
```

*When you feel ready, upload to the production PyPI --*

```
twine upload dist/*
```

You can find this upload at [https://pypi.python.org/pypi/Vespa-Suite] 

It's perfectly OK to upload just one file, e.g. --

```
twine upload dist/Vespa_Suite-0.9.0-py27-none-any.whl
```

If `twine` isn't in your `PATH`, try using `python -m twine` instead of just `twine`.

### Changing Metadata

If you change any of the metadata in `setup.py` (e.g. the description, maintainer email, etc.), you'll need to rerun the `setup.py register` step for that information to appear on PyPI.

If this does not work, then doing a full twine upload will also update any metadata. You can use a throw-away version number like 0.9.dev10 to test this on TestPyPI.

----

## One-time Setup Steps
These are steps that only need to be done once, not every time you upload a wheel.

### Install twine
Because Python tools are not where they need to be yet, we need to rely on a 3rd party package called twine. Install it like so --

```
pip install twine
```

### Create the PyPI Config File
Create the file `.pypirc` in your home directory and populate it with the text below, replacing the stuff in <angle brackets>.

```
[distutils]
index-servers=
    pypi
    testpypi

[testpypi]
repository = https://test.pypi.org/legacy
username = <your test user name goes here>

[pypi]
repository = https://upload.pypi.org/legacy
username = <your production user name goes here>
```

You can also include your password in that file, but that means your password will be stored in plain text on your hard drive. I cannot recommend this.

If you don't put your password in .pypirc, you'll be prompted for it when you run `setup.py register` or use `twine` to upload wheels.

### Register the Project

Register on the test PyPI first --

```
python setup.py register -r test
```

If that goes well, you can register on the production (real) PyPI --

```
python setup.py register
```
