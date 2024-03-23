# DuckDB: `lag` preceding day

The [`lag` window function](https://duckdb.org/docs/sql/window_functions.html#general-purpose-window-functions)
allows to evaluate an `expression` at `offset` rows before the current row.
For time series data, sometimes it is desired to only get previous values in a
certain time window. In the example below, only the entry of the preceding day
should be considered - if it exists.

```sql
DROP TABLE IF EXISTS example;
CREATE TABLE example (
    user_id INTEGER NOT NULL,
    created_at DATE NOT NULL,
    something TEXT
);
INSERT INTO example (user_id, created_at, something) VALUES
    (1, '2024-02-27', 'foo'),
    (1, '2024-02-28', NULL),
    (1, '2024-02-29', 'bar'),
    (2, '2024-02-29', 'foo'),
    (2, '2024-03-01', 'bar'),
    (3, '2024-03-01', 'bar'),
    (4, '2024-03-02', NULL),
    (4, '2024-03-03', 'foo'),
    (4, '2024-03-05', 'bar')
;

SELECT * FROM example;

-- Suppose we want to identify the first day where `something`
-- is 'bar' and the value for `something` on the preceding day,
-- if - and only if - the preceding day has been recorded. We
-- would expect the following result:
--
-- user     created_at      something       something_before
-- 1        2024-02-29      bar             NULL
-- 2        2024-03-01      bar             foo
-- 3        2024-03-01      bar             NULL
-- 4        2024-03-05      bar             NULL

-- The following query almost works:

SELECT * FROM(
    SELECT
        *,
        LAG(something) OVER(
            PARTITION BY user_id
            ORDER BY user_id, created_at
        ) AS something_before
    FROM example
    ORDER BY user_id
)
WHERE
    something = 'bar'
;

-- We get the expected result for user ID `1`, `2` and `3` but
-- also `something_before` with `foo` for the user with ID `4`
-- where no entry has been recorded for the preceding day and
-- `lag()` resolves to the value from 2 days ago.
-- By calculating a `previous_day_offset` we can limit 'lag()'
-- to users with an actual record for the preceding day:

DROP TABLE IF EXISTS example_with_day_offsets;
CREATE TABLE example_with_day_offsets AS (
    SELECT
        *,
        created_at - LAG(created_at) OVER(
            PARTITION BY user_id
            ORDER BY user_id, created_at
        ) AS previous_day_offset
    FROM example
    ORDER BY user_id, created_at
);

SELECT * FROM example_with_day_offsets;

DROP TABLE IF EXISTS example_with_something_before;
CREATE TABLE example_with_something_before AS (
    SELECT
        user_id,
        created_at,
        something,
        CASE
            WHEN
                previous_day_offset = 1
            THEN
                LAG(something) OVER(
                    PARTITION BY user_id
                    ORDER BY user_id, created_at
                )
            ELSE
                NULL
        END AS something_before
    FROM example_with_day_offsets
    ORDER BY user_id, created_at
)
;

SELECT
    *
FROM
    example_with_something_before
WHERE
    something = 'bar'
;
```
