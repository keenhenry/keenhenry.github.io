---
layout: post
title: 'Down the SQLite Rabbit Hole: Debugging FTS5 Logic in a Python Recipe App'
date:   2026-04-01
categories: [technology, programming, web, database, notes]
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


## TL;DR and TIL

- With Python 3.13, you have access to SQLite's powerful extensions (like [`json1`][json1] and [`fts5`][fts5]) by default.
- Do not forget to `COMMIT` transactions on SQLite's [virtual tables][virtual-table].
- Be careful storing [`UUID`][uuid]s: make sure the format is consistent across the application, the easiest way is to have
  a centralized normalization function, and always normalize `UUID`s before storing data.
- For multi-language, user-friendly FTS search support, you may need to consider different [tokenizers][tokenizer]
  and various [FTS5 query syntax][query-syntax] in SQLite. In particular, I found [**prefix queries**][prefix] and
  [**boolean operators**][boolean] useful in providing more user-friendly search experience, while [`jieba`][jieba]
  tokenizer is used to support tokenizing east Asian languages.


## Setting Up FTS with SQLite and Python

In the past, Python's `sqlite3` module in the standard library did not support useful SQLite extensions like `json1` and `fts5`
enabled by default. To use these powerful additions, you either compile (with some compile-time options enabled, like `-DSQLITE_ENABLE_FTS5`)
and build the amalgamation version of SQLite together with your project, or, compile and build the *extension modules* (like `json1.so`
or `fts5.so`) and load them at run-time with your application.

Fortunately, these tedious steps no longer needed if you're using a ['newer' version][fts5-python] (`3.10`+) of Python. The `sqlite3`
module in Python `3.13` was built with SQLite `3.47` together with `json1` and `fts5` extensions by default. In other words, you can
access the powerful, extended features directly by using the *newer versions* of Python 3!

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

        print("Data in 'recipe_fts':")
        results = session.connection().execute(text('SELECT * FROM recipe_fts')).fetchall()
        for r in results:
            print(f'- {r.name} ({r.style})')
```

`create_recipe` is a function defined in the service layer of the application. It is doing exactly what it says:
creating a recipe. In addition to creating a recipe data in *regular* SQLite tables, this function also **upserts**
corresponding recipe data in the `recipe_fts` virtual table.

That means, if everything works, we should see data in the `recipe_fts` table after running this code.

```bash
$ uv run fts.py
Data in 'recipe_fts':
```

Uhoh ... that's not the case. There are bugs in this function already!

After some more poking with the code, I realized that I forgot to **commit** the transactions applied on the FTS table.
It was my first time using [`SQLModel`][sqlmodel] library, I did not forget to commit transactions on regular
SQLite tables, but somehow forgot to do the same on FTS tables.

The fix is easy. Just commit:

```python
session.commit()
```

Now if I run the code above again, I can see data in the `recipe_fts` table.

```bash
$ uv run fts.py
Building prefix dict from the default dictionary ...
Loading model cost 0.664 seconds.
Prefix dict has been built successfully.
Data in 'recipe_fts':
- Garlic   Chicken (Asian)
- Tomato   Pasta (Italian)
```


### Second Bug

Cool, let's do the FTS search via UI. Ooops, no search results found! Now what?

![Not Found](assets/img/20260401/empty-search-results.png){: .normal }
_Empty Search Results_

Apparently, there are problems with FTS search functionality. Let's debug by exercising the FTS search
function `search_recipes_fts` defined in the service layer of the application:

```python
    ...
    # Code omitted ...

    with Session(engine) as session:
        create_recipe(session, recipe1, device_id='test-device')
        create_recipe(session, recipe2, device_id='test-device')

        print("Data in 'recipe_fts':")
        results = session.connection().execute(text('SELECT * FROM recipe_fts')).fetchall()
        for r in results:
            print(f'- {r.name} ({r.style})')
        print()

        print("Search results for 'chicken':")
        results = search_recipes_fts(session, 'chicken')
        for r in results:
            print(f'- {r.name} ({r.style})')

        print("\nSearch results for 'pasta':")
        results = search_recipes_fts(session, 'pasta')
        for r in results:
            print(f'- {r.name} ({r.style})')
```

```bash
$ uv run fts.py
Building prefix dict from the default dictionary ...
Loading model cost 0.664 seconds.
Prefix dict has been built successfully.
Data in 'recipe_fts':
- Garlic   Chicken (Asian)
- Tomato   Pasta (Italian)

Search results for 'chicken':
Search results for 'pasta':
```

It cannot find any recipes in the index?! Strange. How is the FTS search performed in the code?

```python
stmt = select(RecipeTable).from_statement(
    text("""
    SELECT r.*
      FROM recipe_fts f
      JOIN recipes r ON r.id = f.recipe_id
     WHERE recipe_fts MATCH :query
  ORDER BY bm25(recipe_fts)
    """).params(query=query)
)
```

I noticed the regular SQLite table `recipes` and the FTS table `recipe_fts` are **join**ed on `id` and `recipe_id`,
respectively. Then I checked their ID columns in the database:

```bash
sqlite> select id from recipes;
id
--------------------------------
123e4567e89b12d3a456426614174000
1acd71bf971b4b2984ebd78c33d4f465
29f993016d41474f8338786f1ed7127b
749ff44fd8b249aab965aee9a391118d
7f5484009cdd4de68b952048dedbaf32
b2f2fb808bb34a1a8067c0637c49d519
de0a8133759f44b3904212ea2e74acf6
```

and

```bash
sqlite> select recipe_id from recipe_fts;
recipe_id
--------------------------------
7f548400-9cdd-4de6-8b95-2048dedbaf32
b2f2fb80-8bb3-4a1a-8067-c0637c49d519
749ff44f-d8b2-49aa-b965-aee9a391118d
de0a8133-759f-44b3-9042-12ea2e74acf6
```

Boom! The IDs are mismatched! That's why no data in the search results! It turned out the `UUID4`s I stored
in the database had inconsistent formats! In the FTS table, I forgot to 'normalize' the IDs.

The fix is also straight-forward:

```python
id=normalize_uuid(recipe.id),
```

After the fix, the same code run just fine:

```bash
$ uv run fts.py
Building prefix dict from the default dictionary ...
Loading model cost 0.664 seconds.
Prefix dict has been built successfully.
Data in 'recipe_fts':
- Garlic   Chicken (Asian)
- Tomato   Pasta (Italian)

Search results for 'chicken':
- Garlic Chicken (Asian)

Search results for 'pasta':
- Tomato Pasta (Italian)
```

Jubie! Now let me try the search in the UI:

![Found](assets/img/20260401/search-english-found.png){: .normal }
_Search Results in English_


### Third Bug (sort of)

FTS search is working! Now let me try one more thing: search with Chinese characters. I need this functionality
because I have recipes data in Chinese.

![Not Found](assets/img/20260401/chinese-search-not-found.png){: .normal }
_Empty Search Results in Chinese_

What?! The search returns empty results! Why?

After some researching and the help from Google's Gemini, I realized that I need a different tokenizer for building
the FTS index. And one of the tools suggested by Gemini was [`jieba`][jieba] tokenizer for eastern Asian languages.

Please note that `jieba` is an *application*-level tokenizer, not the [built-in tokenizer][tokenizer] in the FTS5
extension in SQLite. So I had to implement some code to tokenize the text before inserting into FTS index:

```python
def normalize_text_for_fts(text: str) -> str:
    """Normalize input text by using jieba to tokenize Chinese text
    and removing punctuation, while also collapsing multiple spaces into one.
    """

    if not text:
        return ''

    tokens = jieba.cut(text)
    result = ' '.join(tokens)

    # Remove punctuation that might mess up FTS5 syntax
    result = re.sub(r'[^\w\s]', ' ', result.strip())
    return result
```

and in function `upsert_recipe_fts`, I adapted the `INSERT` statement:

```python
    session.connection().execute(
        text("""
        INSERT INTO recipe_fts (recipe_id, name, style, category, ingredients_text, instructions_text)
             VALUES (:id, :name, :style, :category, :ingredients, :instructions)
        """),
        {
            'id': recipe.id,
            'name': normalize_text_for_fts(recipe.name),
            'style': normalize_text_for_fts(recipe.style),
            'category': normalize_text_for_fts(recipe.category),
            'ingredients': normalize_text_for_fts(ingredients_text),
            'instructions': normalize_text_for_fts(instructions_text),
        },
    )
```

In addition to using `jieba` tokenizer, I also augment the user input search query to make it more flexible.
To be specific, I incorporate [**prefix queries**][prefix] and [**boolean operators**][boolean] in the query
string to make the search functionlaity more user-friendly.

Now I can also search for Chinese characters:

![Found](assets/img/20260401/chinese-search-found.png){: .normal }
_Search Results in Chinese_

Now my recipe management app has fully functional FTS search functionality 🎉🎉🎉


[elm-nicegui]: https://keenhenry.github.io/posts/elm-architecture-in-nicegui/
[sqlmodel]: https://sqlmodel.tiangolo.com
[jieba]: https://github.com/fxsjy/jieba
[fts5-python]: https://tech-insider.org/sqlite-python-tutorial-fts5-wal-mode-2026/#toc-8
[json1]: https://sqlite.org/json1.html
[fts5]: https://sqlite.org/fts5.html
[virtual-table]: https://sqlite.org/vtab.html
[uuid]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[tokenizer]: https://sqlite.org/fts5.html#tokenizers
[query-syntax]: https://sqlite.org/fts5.html#full_text_query_syntax
[prefix]: https://sqlite.org/fts5.html#fts5_prefix_queries
[boolean]: https://sqlite.org/fts5.html#fts5_boolean_operators