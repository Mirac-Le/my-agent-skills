# PostgreSQL Client API Reference

Detailed guide for using a PostgresClient implementation with psycopg3 + ConnectorX + Polars.

## Table of Contents

1. [Client Initialization](#client-initialization)
2. [Schema Discovery](#schema-discovery)
3. [Reading Data](#reading-data)
4. [Writing Data](#writing-data)
5. [SQL Composition](#sql-composition)
6. [Anti-Patterns](#anti-patterns)
7. [Performance Tips](#performance-tips)

---

## Client Initialization

### Direct Instantiation

```python
from your_package.clients.postgres import PostgresClient

client = PostgresClient(
    "postgresql://user:pass@host:5432/dbname",
    pool_size=5,
    max_overflow=10,
)
```

### Connection Lifecycle

```python
# Client uses connection pooling - no need to manage connections manually
# Close only when completely done (e.g., application shutdown)
client.close()
```

---

## Schema Discovery

**Always inspect table structure before writing queries.**

### List All Tables

```python
tables_df = client.get_tables()
# Columns: table_name, schema_name, table_owner
```

### Get Column Information

```python
info_df = client.get_table_info("public", "MarketValue")
# Columns: column_name, data_type, column_default, is_nullable
```

### Example Output

```
┌─────────────┬───────────┬────────────────┬─────────────┐
│ column_name │ data_type │ column_default │ is_nullable │
├─────────────┼───────────┼────────────────┼─────────────┤
│ id          │ integer   │ nextval(...)   │ false       │
│ trade_day   │ date      │ null           │ false       │
│ fund_code   │ text      │ null           │ false       │
│ nav         │ numeric   │ null           │ true        │
└─────────────┴───────────┴────────────────┴─────────────┘
```

---

## Reading Data

### Method Selection Guide

| Method | Use Case | Performance |
|--------|----------|-------------|
| `read_table()` | Time-series data with date range | High (ConnectorX) |
| `query()` | Complex SQL, joins, aggregations | High (ConnectorX) |
| `query_arrow()` | Need PyArrow Table output | High (ConnectorX) |

### read_table() - Time-Series Data

Best for tables with a timestamp column. Returns data ordered by time.

```python
from datetime import date

df = client.read_table(
    schema="public",
    table="MarketValue",
    start_time=date(2024, 1, 1),   # Inclusive
    end_time=date(2024, 12, 31),   # Exclusive
    ts_column="trade_day",          # Default
    where_clause="fund_account = '12345'",  # Optional filter
    columns=["trade_day", "nav", "total_assets"],  # Optional column selection
)
```

### query() - Custom SQL

For complex queries. Use `psycopg.sql` module for dynamic parts.

```python
# Simple static query (OK for testing)
df = client.query('SELECT * FROM public."FundAccount" WHERE active = true')

# Parameterized query (safe from SQL injection)
df = client.query(
    "SELECT * FROM public.\"MarketValue\" WHERE fund_code = %(code)s",
    {"code": "FUND001"}
)

# Dynamic table/column names - use sql module
from psycopg import sql

q = sql.SQL("SELECT {cols} FROM {schema}.{table}").format(
    cols=sql.SQL(", ").join(map(sql.Identifier, ["trade_day", "nav"])),
    schema=sql.Identifier("public"),
    table=sql.Identifier("MarketValue"),
)
df = client.query(q)
```

---

## Writing Data

### Method Selection Guide

| Method | Use Case | Performance |
|--------|----------|-------------|
| `copy_from_df()` | Bulk insert, append-only | ~10-100x faster |
| `upsert_df()` | Insert or update on conflict | Fast (COPY + ON CONFLICT) |
| `replace_df()` | Replace time range | Atomic DELETE + INSERT |
| `insert_df()` | Small inserts | Standard (row-by-row) |

### copy_from_df() - High-Performance Bulk Insert

```python
# Binary format (default, fastest)
rows_copied = client.copy_from_df("public", "Results", df)

# CSV format (fallback for complex types)
rows_copied = client.copy_from_df("public", "Results", df, binary=False)
```

#### When to use `binary=False`

**CRITICAL**: Set `binary=False` when target table contains `NUMERIC` columns. Binary format causes `InvalidBinaryRepresentation` error.

| Column Type | binary | Note |
|-------------|--------|------|
| `DOUBLE PRECISION`, `INTEGER`, `TEXT` | `True` (default) | ~30% faster for large datasets |
| **`NUMERIC`** | **`False`** | **Required** - binary format incompatible |
| `JSONB` (complex nested) | `False` | Some structures incompatible |

**Decision rule**: Use `NUMERIC` + `binary=False` for financial data requiring precision (NAV, shares). Use `DOUBLE PRECISION` + `binary=True` for performance-critical bulk loads.

### upsert_df() - Insert or Update

Uses temp table + INSERT ON CONFLICT for atomicity.

```python
rows = client.upsert_df(
    schema="public",
    table="DailyNav",
    df=df,
    conflict_columns=["fund_code", "trade_day"],  # Unique constraint
    update_columns=["nav", "updated_at"],  # Columns to update (default: all non-conflict)
)

# For tables with NUMERIC columns, must use binary=False
rows = client.upsert_df(
    schema="public",
    table="nav_custodian",
    df=df,
    conflict_columns=["fund_abbr", "nav_date"],
    binary=False,  # Required for NUMERIC columns
)
```

### replace_df() - Replace Time Range

Atomic DELETE + INSERT in single transaction. Safe for reprocessing.

```python
client.replace_df(
    schema="public",
    table="MarketValue",
    start_time=date(2024, 1, 1),
    end_time=date(2024, 1, 31),
    df=new_data,
    ts_column="trade_day",
)
```

### execute() - DDL and Non-Query Operations

```python
client.execute("TRUNCATE TABLE public.temp_results")
client.execute(sql.SQL("DROP TABLE IF EXISTS {}").format(sql.Identifier("old_table")))
```

---

## SQL Composition

**Critical: Always use `psycopg.sql` module for dynamic SQL parts.**

### Safe SQL Building

```python
from psycopg import sql

# Table/column names → sql.Identifier
table_name = sql.Identifier("MarketValue")
column_name = sql.Identifier("trade_day")

# Values → sql.Literal
date_value = sql.Literal("2024-01-01")

# Combine with sql.SQL
query = sql.SQL("SELECT * FROM {schema}.{table} WHERE {col} = {val}").format(
    schema=sql.Identifier("public"),
    table=table_name,
    col=column_name,
    val=date_value,
)
```

### Common Patterns

```python
# Dynamic column list
cols = ["id", "name", "value"]
cols_sql = sql.SQL(", ").join(map(sql.Identifier, cols))

# Dynamic WHERE clauses
conditions = [
    sql.SQL("{} = {}").format(sql.Identifier("status"), sql.Literal("active")),
    sql.SQL("{} >= {}").format(sql.Identifier("trade_day"), sql.Literal("2024-01-01")),
]
where_clause = sql.SQL(" AND ").join(conditions)
```

---

## Anti-Patterns

### ❌ String Concatenation for SQL

```python
# NEVER do this - SQL injection risk
table = user_input
df = client.query(f"SELECT * FROM {table}")  # Dangerous!
```

### ❌ Row-by-Row Operations

```python
# NEVER do this - extremely slow
for row in data:
    client.insert("public", "Results", [row])
```

### ❌ Business Logic in SQL

```python
# Avoid complex calculations in SQL
df = client.query("""
    SELECT 
        a.*, 
        b.price * a.quantity AS market_value,
        CASE WHEN ... THEN ... END AS adjusted_value
    FROM ... 
    JOIN ...
""")
```

### ✅ Fetch Raw Data, Process in Polars

```python
# Better: fetch base data, compute in Polars
positions = client.read_table("public", "Positions", start, end)
prices = client.read_table("public", "Prices", start, end)

result = (
    positions
    .join(prices, on=["symbol", "trade_day"])
    .with_columns(
        (pl.col("quantity") * pl.col("price")).alias("market_value")
    )
    .filter(pl.col("market_value") > 0)
)
```

---

## Performance Tips

1. **Use `read_table()` with column selection** - Don't fetch columns you don't need
2. **Batch writes** - Accumulate data, write once with `copy_from_df()`
3. **Binary COPY format** - Default for `copy_from_df()` and `upsert_df()` (except NUMERIC)
4. **Connection pooling** - Reuse client instance, don't create new connections per query
