---
layout: post
title:  Elm Architecture and NiceGUI
date:   2026-02-14
categories: [frontend, web, functional programming, technology, notes]
tags: [elm, python, elm archiecture, fp, nicegui]
description: Notes of basic computer network knowledge
---

Recently I learned something useful regarding functional programming language and web frontend
development: [**Elm**][elm] and [**The Elm Architecture** (TEA)][tea].

TEA is a design pattern: `Model` ➡️ `View` ➡️ `Update`, which facilitates uni-directional data flow.
That makes Web UI development *simple* and *clean*. In addition, the 'functional' approach of Elm makes
the Web UI components easier to test (I like 😉). For more details about TEA, you can refer to
[this diagram here][tea-explanation].

Having learned this technique, I couldn't resist to play with it and see how it can transform Web UI
development! So I decided to try it in my personal project, [`recipy`][recipy] (a simple local-first
recipe app).

I want to create this app because I cooked a lot, and I need to refer to my personal recipes from
time to time (as I couldn't remember all the details, and I have a lot of recipes (`> 100`)). I need
some way to manage the information and search/reference recipes easily and quickly while
cooking.

In the meantime, I was curious about a Python UI library, [`NiceGUI`][nicegui], it seems to be a
very convenient component-based Web UI development framework which lets your do Web UI development
completely in Python (no JS, CSS and HTML)!

So I decided to try `recipy` with `NiceGUI` and applying the ideas from **TEA**.

Before documenting all the details of what I did, some high level design decision first:

1. `recipy` is a simple recipe management app, it doesn't have complicated UIs and states, so I intentionally kept state management simple
   and kept state only in the database. The choice of database is **SQLite** because it is *a private, local-first* recipe library, not a service.
   The database is the single source of truth. That means the design already deviates from TEA. In other words, I apply TEA only partially
   for this project. That also means instead of `Model` ➡️ `View` ➡️ `Update`, you only see `View` ➡️ `Update`, since I don't need to
   keep/manage state/model in the Web app.

```python
-- View
-- Update
```

2. I chose to implement a MPA instead of a SPA. This is also a natural consequence of decision 1, state management is only in database.
   Note: in **NiceGUI**, it is possible to implement SPA via [`subpages`][nicegui-subpages], otherwise, normal pages are MPA pages.


[elm]: https://guide.elm-lang.org
[tea]: https://guide.elm-lang.org/architecture/
[tea-explanation]: https://sporto.github.io/elm-workshop/03-tea/01-intro.html
[recipy]: https://gitlab.com/keenhenry/recipy
[nicegui]: https://nicegui.io
[nicegui-subpages]: TODO
