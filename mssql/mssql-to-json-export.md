# MSSQL to JSON export

Sometimes I need to export data from a MSSQL server to JSON.

```
./fetch-data.py MyDB.dbo.MyTable MyTable.json
```

Requires the [Microsoft ODBC driver](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver16&tabs=redhat18-install%2Calpine17-install%2Credhat7-install%2Credhat7-13-install%2Crhel7-offline), [pyodbc](https://github.com/mkleehammer/pyodbc)
and depending on the exported data a different [`default` function](https://docs.python.org/3/library/json.html#basic-usage).

```python
#!/usr/bin/env python3

import decimal
import json
import os
import sys
from datetime import datetime, date

import pyodbc


def db_connect():
    host = os.environ["MSSQL_HOST"]
    username = os.environ["MSSQL_USER"]
    password = os.environ["MSSQL_PASS"]

    # https://github.com/mkleehammer/pyodbc/wiki/Connecting-to-SQL-Server-from-Windows

    driver = "{ODBC Driver 18 for SQL Server}"

    conn_str = f"DRIVER={driver};SERVER={host};UID={username};PWD={password};ENCRYPT=yes;TrustServerCertificate=yes;ApplicationIntent=readonly"

    return pyodbc.connect(conn_str)


argv = sys.argv[1:]
if len(argv) != 2:
    print("Usage: %s <MSSSQL DB table identifier> <path/to/file.json>" % sys.argv[0])
    print("Example: %s <MyDB.dbo.MyTable> <data/MyTable.json>" % sys.argv[0])
    sys.exit(1)

src_table = argv[0]
dst_json_file = argv[1]

print("Checking connection ...")

conn = db_connect()
cursor = conn.cursor()

cursor.execute("SELECT @@version;")
row = cursor.fetchone()
while row:
    print(row[0])
    row = cursor.fetchone()

print("Success")

print("Fetching table data ...")

cursor.execute("SELECT * FROM %s" % src_table)

columns = [column[0] for column in cursor.description]

data = []

for row in cursor.fetchall():
    data.append(dict(zip(columns, row)))


def _serialize(obj):
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    if isinstance(obj, decimal.Decimal):
        return str(obj)
    raise TypeError("%s is not JSON serializable" % type(obj))


with open(dst_json_file, "w", encoding="utf8") as f:
    json.dump(data, f, indent=2, sort_keys=True, default=_serialize)

print("Done")
```
