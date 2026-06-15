---
layout: post
title: Python Error Hanling Nuances
date:   2026-04-01
categories: [technology, programming, notes]
tags: [error handling, python, TIL]
description: raise vs. raise e vs. raise e from ex
---

## The Question

Exception handling is a common programming construct many programming languages. In Python, the `try` statement is used to
capture and handle Python exceptions. The `except` clause inside the `try` statement is used to define the **exception handler**.
And, a lot of times, we simply **reraise** the exception for _higher level_ code to handle it.

To reraise, we have 3 difference ways of using `raise` statement:


### `raise e`

```python
try:
    ...  # Nasty error occurs!
except SomeNastyError as e:
    print(f"Caught an error {e}")
    raise e
```


### `raise`

```python
try:
    ...  # Nasty error occurs!
except SomeNastyError as e:
    print(f"Caught an error {e}")
    raise
```


### `raise e from ex`

```python
try:
    ...  # Nasty error occurs!
except SomeNastyError as e:
    print(f"Caught an error {e}")
    raise SomeOtherCustomError('User-faceing error message') from e
```

What is the difference? And when to use what?


## The Answer

Notice that the first `raise` is followed by the captured exception `e`, while the second `raise` not.

The first version reraise the *same* exception object but also *reset* the traceback (the `__traceback__`
attribute of the exception object) to include the `raise e` line in addition to the original line that
caused `SomeNastyError` in the `try` block. So, it adds extra noise/information to the original exception.

The second version simply **preserves** the orginal traceback and propagate the exception to the handler code
higher up in the call stack. The error and traceback message is clean and points to the original 'cause'
of the problem directly. In other words, if your *intention is to 'simply reraise'* the original exception,
**this version is WHAT YOU WANT**, not the first version.

The third version is [**exception chaining**][pep3134] feature that became available in Python 3. This
gives the programmer a tool to intentionally relate two exceptions together, with the original exception (in
the `from` clause) being the 'cause' of the second exception. This is useful to translate low-level
exception into higher level domain error which is more presentable to the end-user.

Another common pattern of the exception chaining features is to `raise EXCEPTION from None`, this is equivalent to:

```python
exc = EXCEPTION
exc.__cause__ = None
raise exc
```

This suppresses the original context, hiding the 'cause' exception from the user; this is useful when
you want to hide the implementation details from the user.


## Summary

To reraise exceptions, most of the time you only need **version 2**: `raise`. Do not use **version 1**.
Version 3 can be useful when you want to change abstraction level and preserve information in between
the change of abstraction.

Some relevant reference is [here][raise-vs-raise-e].


[raise-vs-raise-e]: https://dev.to/kfir-g/raising-the-difference-between-raise-and-raise-e-378h
[pep3134]: https://peps.python.org/pep-3134/