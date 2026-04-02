TODO

Things to mention:

- python 3.13 sqlite3 library already includes FTS5 extension/functionality, so no need to FTS5 load extension at run-time!
- First bug discovered: `uuid` presentation mismatch between `recipes` table and `recipe_fts` table - `str(uuid)` vs. uuid stored automatically by sqlalchemy.
- Second bug discovered: inserts/deletes/updates on FTS tables are not **committed**, fogot to do so. Fix: `session.commit()`
- Third bug discovered: default tokenizer support does not support chinese language very well, had to find a dedicated tokenizer for chinese language `jieba`; this solved all the problems.
