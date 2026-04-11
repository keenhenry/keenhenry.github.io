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

Based on my past experience, the `sqlite3` module in Python's standard library does not support some useful extensions like `json1` and `fts5`
by default. To use these useful features, you either need to TODO (custom-compiled almaglamation? version) or use some extenstions and load them
as `.so` files at run-time with Python blablabl.

However, surprisingly,

## FTS table Schema
TODO

Things to mention:

- python 3.13 sqlite3 library already includes FTS5 extension/functionality, so no need to FTS5 load extension at run-time!
- First bug discovered: `uuid` presentation mismatch between `recipes` table and `recipe_fts` table - `str(uuid)` vs. uuid stored automatically by sqlalchemy.
- Second bug discovered: inserts/deletes/updates on FTS tables are not **committed**, fogot to do so. Fix: `session.commit()`
- Third bug discovered: default tokenizer support does not support chinese language very well, had to find a dedicated tokenizer for chinese language `jieba`; this solved all the problems.


[elm-nicegui]: https://keenhenry.github.io/posts/elm-architecture-in-nicegui/
