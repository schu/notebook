# DuckDB: aggregate multiple rows into a single JSON object

The other day I was looking for a way to transform a table with multiple rows
per user to a single row per user layout. The source table had a structure
similar to the following example:

```sql
CREATE TABLE example (
    user_id INTEGER NOT NULL,
    item_id INTEGER NOT NULL,
    value1 TEXT,
    value2 TEXT
);
```

Depending on `item_id`, `value1` and `value2` have different meanings and
expected values. While this layout potentially does reduce used space and
avoids a high number of columns, it makes querying and understanding the data
more difficult and error prone and I'm sceptical about if the benefits do
outweigh the costs. Instead, I wanted a single row per user with all data in a
JSON object. The data could then be queried with DuckDB's [JSON scalar
functions](https://duckdb.org/docs/extensions/json#json-scalar-functions).

For example:


```sql
DROP TABLE IF EXISTS example;
CREATE TABLE example (
    user_id INTEGER NOT NULL,
    item_id INTEGER NOT NULL,
    value1 TEXT,
    value2 TEXT
);
INSERT INTO example (user_id, item_id, value1, value2) VALUES
    (1, 1000, 'foo', NULL),
    (1, 2000, 'bar', 23),
    (2, 2000, 'bar', 42),
    (3, 1000, NULL, NULL),
    (3, 3000, 123, 'baz'),
    (3, 3000, 456, 'baz')
;

-- Instead of a row per item_id transform into a row per user_id
-- with all items collected into a JSON object

DROP TABLE IF EXISTS example_transformed;
CREATE TABLE example_transformed AS (
    SELECT
        user_id,
        -- A map key (here `item_id`) has to be unique, but the
        -- example above contains duplicate item IDs (here `3000`)
        -- for a single user ID (here `3`), therefore we first
        -- have to collect all item values into an array in the
        -- inner select statement before we can group the items
        -- into a column `items`
        JSON_GROUP_OBJECT(
            item_id, values
        ) AS items
    FROM (
        SELECT
            user_id,
            item_id,
            JSON_GROUP_ARRAY(
                JSON_OBJECT('value1', value1, 'value2', value2)
            ) AS values
        FROM example
        GROUP BY user_id, item_id
    )
    GROUP BY user_id
);

-- Now we can select specific users based on specific items and values
-- using DuckDB's JSON scalar functions, for example:
SELECT
    *
FROM
    example_transformed
WHERE
    -- All rows with an object with key "1000" where value1 is not null
    JSON_CONTAINS(items, '{"1000": []}')
        AND NOT JSON_CONTAINS(items, '{"1000": [{"value1": null}]}')
;
```
