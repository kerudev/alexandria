# big_query/queries

Databases are one of the core utilities I use daily as a backend developer.

Even though SQL is fairly simple to understand, I find it hard to remember all
the powerful queries I use sometimes (90% of my queries are SELECT statements).

This document will store those queries, ranging from simple to complex.

Big Query is the engine I'm using the most right now. The following queries
might need a few tweaks to work on other engines.

## Copy table

Copying tables is useful to run tests without affecting the original data.

Imagine your client is connected to a table and you accidentally delete half of
it. Now imagine that table had two years worth of data and you need to quickly 
reprocess it before your client notices (some of us don't need to imagine...).

There are two ways of achieving this:

```sql
CREATE TABLE `...` 
COPY `...`;
```

```sql
CREATE TABLE `...`
CLONE `...`;
```

Both have their use cases. `COPY` is better for backups, snapshots and moving
data to other projects or (compatible) regions, while `CLONE` is better for
testing and temporary tables.

The key difference between `COPY` and `CLONE` is that `COPY` does a full copy
of the original table, while `CLONE` follows a copy-on-write philosophy.

On the pricing side, `CLONE` is cheaper than `COPY`, as it shares the same
physical records (bytes on a disk) as the original table. `CLONE` is also
faster on great volumes of data.

There is a caveat with `CLONE`, being that you can't have more than 3 chaines
clones, or what is the same:

```sql
CREATE TABLE `clone_1` CLONE `original`;  -- ok
CREATE TABLE `clone_2` CLONE `clone_1`;   -- ok
CREATE TABLE `clone_3` CLONE `clone_2`;   -- ok
CREATE TABLE `clone_4` CLONE `clone_3`;   -- fails
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/table-clones-intro
- https://docs.cloud.google.com/bigquery/docs/table-clones-create
