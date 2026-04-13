---
layout: post
title: 'Implementing Recipe Search Logic: A Deep Dive into Python and SQLite FTS5'
date:   2026-04-01
categories: [programming, web, database, technology, notes]
tags: [fts, sqlite, sqlmodel, python]
description: My step-by-step journey of implementing Full-Text Search (FTS) in a Python recipe management app, from resolving library errors
             to ranking results with BM25.
---


## Background

As I mentioned in [this post][elm-nicegui], I have been building a recipe management app for myself and other potential users. In this journey,
I have already discovered and learned many things. This is one of the posts to document my findings and learnings.

For prototyping purpose, I have been using **Python** `3.13` as the main programming language, and **SQLite** as the main data storage.

One of the MVP features I am implementing is searching for recipes, in particular, I am implementing FTS (Full Text Search). To achieve that,
I am using SQLite's FTS5 extension.


## Setting Up FTS with SQLite and Python

Based on my past experience, the `sqlite3` module in Python's standard library does not support some useful SQLite extensions like `json1` and `fts5`
out of the box. To use these useful additions, you either need to compile (with some compile-time options enabled, like `-DSQLITE_ENABLE_FTS5`) and build
the amalgamation version of SQLite link with your project, or, compile and build the extension modules (like `json1.so` or `fts5.so`) and load them at
run-time with your application.

Fortunately, these tedious steps no longer needed if you're using 'newer' version (`3.10`+) of Python. The `sqlite3` module in Python `3.13`'s standard
library bundles with SQLite `3.47` and with `json1` and `fts5` support by default.

To be sure, I tried the following in a SQLite **in-memory** database in Python:

```python
TODO
```


## FTS5 Table Schema

TODO

After running such code, I can see that the FTS5 table was created successfully in the database.


## First Bug

However, after adding some recipes via the UI of the application, the FTS table is still empty. What now?

- First bug discovered: `uuid` presentation mismatch between `recipes` table and `recipe_fts` table - `str(uuid)` vs. uuid stored automatically by sqlalchemy.

## Second Bug

After fixing `uuid` format problem, the FTS table is still empty. What now?
- Second bug discovered: inserts/deletes/updates on FTS tables are not **committed**, fogot to do so. Fix: `session.commit()`

## Third Bug (sort of)

Now the FTS table is not empty! But, the FTS search with Chinese characters still returns empty result! What now?

- Third bug discovered: default tokenizer support does not support chinese language very well, had to find a dedicated tokenizer for chinese language `jieba`; this solved all the problems.



[elm-nicegui]: https://keenhenry.github.io/posts/elm-architecture-in-nicegui/
