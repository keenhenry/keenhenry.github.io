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

TEA is a design pattern: `Model` ➡️ `View` ➡️ `Update`. That makes Web UI development *simple* and *clean*.
I really like the idea and its 'functional' approach, which means the Web UI becomes easier to test,
I like 😉. For more details about TEA, you can refer to [this diagram here][tea-explanation].

Having learned this technique, I couldn't resist to play with it and see how it can transform Web UI
development! So I decided to try it in my personal project, [`recipy`][recipy] (a simple local-first
recipe app).

I want to create this app because I cooked a lot, and I need to refer to my personal recipes from
time to time (as I couldn't remember all the details, and I have a lot of recipes (`> 100`)). I need
some way to manage the information and let me search/reference recipes easily and quickly while
cooking.

In the meantime, I was curious about a Python UI library, [`NiceGUI`][nicegui], it seems to be a
very convenient component-based Web UI development framework which lets your do Web UI development
completely in Python (no JS, CSS and HTML)!

So I decided to try `recipy` with `NiceGUI` and in TEA. Here is what I found:




[elm]: https://guide.elm-lang.org
[tea]: https://guide.elm-lang.org/architecture/
[tea-explanation]: https://sporto.github.io/elm-workshop/03-tea/01-intro.html
[recipy]: https://gitlab.com/keenhenry/recipy
[nicegui]: https://nicegui.io