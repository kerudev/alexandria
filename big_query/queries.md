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

## Missing dates

This one is super useful for me, as I work with tables that save data everyday.
Sometimes, there are dates missing (APIs not returning a response, failed
reprocesses, accidental deletion...).

This query returns the dates that are not present inside a range.

```sql
WITH params AS (
  SELECT
    DATE '2024-01-01' AS start_date, 
    DATE '2026-06-01' AS end_date
),
all_dates AS (
  SELECT start_date + INTERVAL n DAY AS purchase_date
  FROM params,
  UNNEST(GENERATE_ARRAY(0, DATE_DIFF(end_date, start_date, DAY))) AS n
),
dates AS (
  SELECT DISTINCT purchase_date
  FROM `...`
  WHERE DATE(purchase_date) BETWEEN (SELECT start_date FROM params)
                        AND (SELECT end_date FROM params)
    AND item = "bike"  -- optional extra filters
)
SELECT ad.purchase_date
FROM all_dates ad
LEFT JOIN dates d ON ad.purchase_date = DATE(d.purchase_date)
WHERE d.purchase_date IS NULL
ORDER BY ad.purchase_date;
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#with_clause 
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#unnest_operator
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/array_functions#generate_array

## Pivot table

As the docs say: `PIVOT` operator rotates rows into columns, using aggregation.

It's hard for me to explain pivoting without an example to visualize it, so I
copied an example from the docs:

```sql
-- Before PIVOT is used to rotate sales and quarter into Q1, Q2, Q3, Q4 columns:
/*---------+-------+---------+------+
 | product | sales | quarter | year |
 +---------+-------+---------+------|
 | Kale    | 51    | Q1      | 2020 |
 | Kale    | 23    | Q2      | 2020 |
 | Kale    | 45    | Q3      | 2020 |
 | Kale    | 3     | Q4      | 2020 |
 | Kale    | 70    | Q1      | 2021 |
 | Kale    | 85    | Q2      | 2021 |
 | Apple   | 77    | Q1      | 2020 |
 | Apple   | 0     | Q2      | 2020 |
 | Apple   | 1     | Q1      | 2021 |
 +---------+-------+---------+------*/

-- After PIVOT is used to rotate sales and quarter into Q1, Q2, Q3, Q4 columns:
/*---------+------+----+------+------+------+
 | product | year | Q1 | Q2   | Q3   | Q4   |
 +---------+------+----+------+------+------+
 | Apple   | 2020 | 77 | 0    | NULL | NULL |
 | Apple   | 2021 | 1  | NULL | NULL | NULL |
 | Kale    | 2020 | 51 | 23   | 45   | 3    |
 | Kale    | 2021 | 70 | 85   | NULL | NULL |
 +---------+------+----+------+------+------*/
```

The query of that looks something like this:

```sql
SELECT *
FROM (
  SELECT product, sales, quarter, year
  FROM `...`
)
PIVOT (
  SUM(sales)
  FOR quarter IN ('Q1', 'Q2', 'Q3', 'Q4')
)
ORDER BY product, year;
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#pivot_operator

## Missing columns

This query is useful when doing unions of tables, which require columns on both
ends to have the same names and types.

```sql
WITH
table_a AS (
  SELECT column_name
  FROM `...`
  WHERE table_name = 'table_a'
),
table_b AS (
  SELECT column_name
  FROM `...`
  WHERE table_name = 'table_b'
)

SELECT
  'A' AS source,
  a.column_name,
  a.data_type
FROM table_a a
LEFT JOIN table_b b
  ON a.column_name = b.column_name
WHERE b.column_name IS NULL

UNION ALL

SELECT
  'B' AS source,
  b.column_name,
  b.data_type
FROM table_b b
LEFT JOIN table_a a
  ON a.column_name = b.column_name
WHERE a.column_name IS NULL
```

## Dynamic queries

Sometimes regular queries are not enough as you might need tje flexibility that
you could have with a regular programming language. That's where the next query
comes into play, as it allows you to dynamically create a query, then execute
it once you are done.

The following query generates a query string that counts how many distinct
values has each column in a table, then executes it. This saves us from writing
each column name on a table manually.

All of this is called `procedural language` in Big Query.

```sql
DECLARE sql STRING;

SET sql = (
  SELECT STRING_AGG(
    FORMAT("COUNT(DISTINCT %s) AS %s_distinct", column_name, column_name),
    ",\n"
  )
  FROM `...`
  WHERE table_name = 'table'
);

SET sql = FORMAT("""
  SELECT
  %s
  FROM `...`
""", sql);

EXECUTE IMMEDIATE sql;
```

Docs:
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/string_functions#format_string
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/aggregate_functions#string_agg
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/procedural-language#declare
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/procedural-language#set
- https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/procedural-language#execute_immediate

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

There is a caveat with `CLONE`, being that you can't have more than 3 chained
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
ON T.purchase_date = S.purchase_date
WHEN NOT MATCHED BY TARGET THEN
  INSERT (
    purchase_date,
    item,
    stock,
    price
  )
  VALUES (
    S.purchase_date,
    S.item,
    S.stock,
    S.price
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
