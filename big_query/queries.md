# big_query/queries

Databases are one of the core utilities I use daily as a backend developer.

Even though SQL is fairly simple to understand, I find it hard to remember all
the powerful queries I use sometimes (90% of my queries are SELECT statements).

This document will store those queries, ranging from simple to complex.

Big Query is the engine I'm using the most right now. The following queries
might need a few tweaks to work on other engines.

## Format floats

Sometimes, floats have too many decimals and we just want the first two.

```sql
SELECT
  FORMAT("%'.2f", SUM(Cost)) AS Cost,
  FORMAT("%'.2f", SUM(Spend)) AS Spend,
  ...
FROM `...`
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#format_string
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#format_specifiers

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

Bonus: you can copy a table from a query result. The benefit of this is that
you can select the fields you want to copy (in case you don't want them all).

```sql
CREATE TABLE `...` AS
SELECT * FROM `...`;
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/table-clones-intro
- https://docs.cloud.google.com/bigquery/docs/table-clones-create
- https://docs.cloud.google.com/bigquery/docs/tables#create_a_table_from_a_query_result

## Merge data

You might find the rare case where you got two tables and you need to copy rows
from one to another, without duplicating the rows.

Imagine you got a production table and a testing table. While running some test
on production (don't do that, kids), you change the destination table and the
testing one is being updated instead of the production one.

Merging is a way to save just the new data back into production.

The following example merges data based on a date column, but you can adapt it
to your use case.

```sql
-- T: target
-- S: source

MERGE `...` AS T
USING `...` AS S
ON T.Day = S.Day
WHEN NOT MATCHED BY TARGET THEN
  INSERT (
    Day,
    Item,
    Stock,
    Price
  )
  VALUES (
    S.Day,
    S.Item,
    S.Stock,
    S.Price
  );
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#merge_statement
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#merge_examples

## Rename column

Renaming columns is useful when you made a mistake with a name or you just want
to change the name of something, but it's a risky operation if someone dependes
on your table: views, Looker or PowerBI dashboards, external consumers...

So keep this in mind if that's your case.

```sql
ALTER TABLE `...`
RENAME COLUMN PRICE TO cost;
```

Big Query is case insensitive about columns names, so if you need to change the
case of one, you'll need to rename it twice.

If you just change the case it will be the same for Big Query, and it will tell
you that you can't rename the column to the same name.

```sql
-- Step 1: temp name
ALTER TABLE `...`
RENAME COLUMN PRICE TO price_tmp;

-- Step 2: final name
ALTER TABLE `...`
RENAME COLUMN price_tmp TO price;
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/managing-table-schemas#change_a_columns_name
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#alter_table_rename_column_statement

## Drop columns

There are two ways to drop columns:

1. Drop them individually:

```sql
ALTER TABLE `...`
DROP COLUMN column_to_delete;
```

2. Overwrite the table by excluding the columns you don't want:

```sql
CREATE OR REPLACE TABLE `...` AS (
  SELECT * EXCEPT (column_to_delete_1, column_to_delete_2, column_to_delete_3) FROM `...`
);
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/managing-table-schemas#delete_a_column
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#alter_table_drop_column_statement
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#select_except
