---
sort: 4
---

# About Python Reference Counting
This document is an overview on Python's reference counting from the
perspective of the C API, with an emphasis on the context of the ICE
Python code.

## Background
As Python programmers we have the luxury of mostly ignoring reference
counting. When working with Python code in C, we lose that luxury, but gain
brackets and semicolons.

Tracking references isn't too hard, but there are some subtleties of which
to be aware.

Of course, the canonical reference for this subject is
[Python's own documentation](https://docs.python.org/2/c-api/intro.html#objects-types-and-reference-counts).
This is just an overview on the subject.

## Kinship With `malloc()` and `free()`
References have some similarities to C memory allocation. Adding a reference
to an object and never removing it is akin to allocating memory and not
freeing it. It's a memory leak, and these silent errors can be difficult to
track. You might not even notice them, if the leak is small enough.

Removing a reference that you don't own is akin to a double-free bug. Results
are undefined, but segfaults are common.

And just as C programs can pass around responsibility for allocated memory,
Python references often change hands over time.

## Stealing and Borrowing Your References
Python's documentation uses "stealing" and "borrowing" to refer to the exchange
of references. Stealing and borrowing (or whatever terms one uses) are
contractual arrangements between functions and their callers. If either party
violates the contract, something bad will happen. (That's either a memory leak
or a double-free which will likely lead to a segfault.)

I prefer different terms to describe these contracts. When a function is
documented to _steal_ the reference passed to it, I don't think of
it as stealing. I think of it as a mutual handoff of responsibility. This
call _steals_ a reference to my object --
```
PyTuple_SetItem(p_tuple, position, p_my_object);
```

I would say that `PyTuple_SetItem()` _takes responsibility for_ the reference to
my object.

Most Python API functions _borrow_ references. This means that they
temporarily use the reference you pass to them, but once the function is
complete they don't use your reference anymore. You retain responsibility
for decrementing the reference count.

