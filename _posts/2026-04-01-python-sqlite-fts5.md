---
layout: post
title: 'Down the SQLite Rabbit Hole: Debugging FTS5 Logic in a Python Recipe App'
date:   2026-04-01
categories: [programming, web, database, technology, notes]
tags: [fts, sqlite, sqlmodel, python]
description: My step-by-step journey of debugging Full-Text Search (FTS) in a Python recipe management app.
---


## Background

As I mentioned in [this post][elm-nicegui], I have been building a recipe management app for myself and other potential users.
In this journey, I have discovered and learned many things. This is one of the posts to document my findings and learnings.

For prototyping purpose, I have been using **Python** `3.13` as the main programming language. In addition, **SQLite** is used
as the main data storage for this **private, local-first** app.

One of the MVP features I am implementing is *searching* for recipes, in particular, I am implementing FTS (Full Text Search).
To achieve that, I am taking advantages of SQLite's FTS5 functionality.


## Setting Up FTS with SQLite and Python

In the past, Python's `sqlite3` module in the standard library did not support useful SQLite extensions like `json1` and `fts5`
by default. To use these powerful additions, you either compile (with some compile-time options enabled, like `-DSQLITE_ENABLE_FTS5`)
and build the amalgamation version of SQLite together with your project, or, compile and build the *extension modules* (like `json1.so`
or `fts5.so`) and load them at run-time with your application.

Fortunately, these tedious steps no longer needed if you're using a 'newer' version (`3.10`+) of Python. The `sqlite3` module in
Python `3.13` was built with SQLite `3.47` together with `json1` and `fts5` extensions by default. In other words, you can access
the powerful, extended features directly by using the *newer versions* of Python 3!

To be sure, I tried the following in a SQLite **in-memory** database in Python:

```python
if __name__ == '__main__':
    # Example service layer usage and testing of FTS functionalities

    from sqlmodel import create_engine, SQLModel

    engine = create_engine('sqlite:///:memory:')
    SQLModel.metadata.create_all(engine)

    with engine.connect() as conn:
        conn.exec_driver_sql("""
        CREATE VIRTUAL TABLE IF NOT EXISTS recipe_fts USING fts5(
            recipe_id UNINDEXED,
            name,
            text
        );
        """)
        conn.commit()
```

If running the code above doesn't crash and a virtual table `recipe_fts` is created
in the database, then the FTS5 feature is surely available for your Python version.


## Debugging Journey

The simple test above assured me my application can use the FTS feature directly. The FTS5 table `recipe_fts` is
visible in the database. However, after adding some recipes via the UI of the application, `recipe_fts` table is
empty, what's going on?


### First Bug

Okay, let's take a step back. Let's make sure I can add data into FTS table with my existing code in the
**in-memory** SQLite database first. By extending the testing code above, I've got the following:


```python
if __name__ == '__main__':
    # Example service layer usage and testing of FTS functionalities

    from sqlmodel import create_engine, SQLModel
    from services import create_recipe, search_recipes_fts
    from models import Recipe

    engine = create_engine('sqlite:///:memory:')
    SQLModel.metadata.create_all(engine)
    create_fts_table(engine)

    # Create test recipe data
    recipe1 = Recipe(
        name='Garlic Chicken',
        style='Asian',
    )

    recipe2 = Recipe(
        name='Tomato Pasta',
        style='Italian',
    )

    with Session(engine) as session:
        create_recipe(session, recipe1, device_id='test-device')
        create_recipe(session, recipe2, device_id='test-device')
```

`create_recipe` is a function defined in the service layer of the application. It is doing exactly what it says:
creating a recipe. In addition to creating a recipe data in *regular* SQLite tables, this function also **upserts**
corresponding recipe data in the `recipe_fts` virtual table.

That means, if everything works, we should see data in the `recipe_fts` table after running this code.

Uhoh ... that's not the case. There are bugs in this function already!

After some more poking with the code, I realized that I forgot to **commit** the transactions applied on the FTS table.
It was my first time using [`SQLModel`][sqlmodel] library, I did not forget to commit transactions on regular
SQLite tables, but somehow forgot to do the same on FTS tables.

The fix is easy. Just commit:

```python
session.commit()
```

Now if I run the code above again, I can see data in the `recipe_fts` table. Cool, let's do the FTS search
via UI. Ooops, no search result found! Now what?


### Second Bug

- Second bug discovered: `uuid` presentation mismatch between `recipes` table and `recipe_fts` table - `str(uuid)` vs. uuid stored automatically by sqlalchemy.

After fixing `uuid` format problem, the FTS table is still empty. What now?

```python
        print("Search results for 'chicken':")
        results = search_recipes_fts(session, 'chicken')
        for r in results:
            print(f'- {r.name} ({r.style})')

        print("\nSearch results for 'pasta':")
        results = search_recipes_fts(session, 'pasta')
        for r in results:
            print(f'- {r.name} ({r.style})')
```

### Third Bug (sort of)

Now the FTS table is not empty! But, the FTS search with Chinese characters still returns empty result! What now?

- Third bug discovered: default tokenizer support does not support chinese language very well, had to find a dedicated tokenizer for chinese language `jieba`; this solved all the problems.


Add this link somewhere in this post: https://tech-insider.org/sqlite-python-tutorial-fts5-wal-mode-2026/#toc-7

[elm-nicegui]: https://keenhenry.github.io/posts/elm-architecture-in-nicegui/
[sqlmodel]: https://sqlmodel.tiangolo.com