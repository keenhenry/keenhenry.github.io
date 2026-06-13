---
layout: post
title: Python Error Hanling Nuances
date:   2026-04-01
categories: [technology, programming, notes]
tags: [error handling, python, TIL]
description: TODO
---

## The Problem

Exception handling is a common programming construct in Python. In particular, the `try ... except ...` statement is used to capture and
handle Python exceptions. In the `except` block, a lot of times we **reraise** the exception for the caller code to handle it.

However, to reraise, we have two difference ways of using `raise`:

```python
try:
    ...  # Nasty error occurs!
except SomeNastyError as e:
    print(f"Caught an error {e}")
    raise e
```

Or, you can do it like this:

```python
try:
    ...  # Nasty error occurs!
except SomeNastyError as e:
    print(f"Caught an error {e}")
    raise
```

What is the difference?


## The Answer

Notice that the first `raise` is followed by the captured exception `e`, while the second `raise` not.

Semantically, what is different?

TODO


## Reference

- [Difference between `raise` and `raise e`][raise-vs-raise-e]


[raise-vs-raise-e]: https://dev.to/kfir-g/raising-the-difference-between-raise-and-raise-e-378h
