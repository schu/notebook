# DuckDB: migrate data with multi-database support

[DuckDB's multi-database-support](https://duckdb.org/2024/01/26/multi-database-support-in-duckdb.html)
can be a convenient way to transform and move data between databases.

For example, with the [SQLite](https://duckdb.org/docs/extensions/sqlite) and
[MySQL extension](https://duckdb.org/docs/extensions/mysql.html):

```sql
ATTACH 'host=HOST port=0 database=DATABASE user=USER password=PASSWORD' AS src (TYPE mysql);

ATTACH 'mydb.sqlite3' AS dst (TYPE sqlite);

WITH import AS (
    SELECT
        t.somecolumn,
        o.othercolumn
        -- ...
    FROM src.mytable t
    INNER JOIN src.othertable o ON o.id = t.id
    WHERE ...
) INSERT INTO dst.postings (
    -- column1
    -- column2
    -- ...
) SELECT
    -- value1
    -- value2
    -- ...
FROM import;
```
