---
name: postgres-client
description: High-performance PostgreSQL client patterns using psycopg3 + ConnectorX + Polars. Use when building Python applications that need efficient database I/O with DataFrame workflows. Covers bulk operations (COPY), upsert patterns, safe SQL composition, and performance optimization.
---

# PostgreSQL Client Patterns

High-performance PostgreSQL interaction patterns for Python applications.

## Tech Stack

| Component | Purpose |
|-----------|---------|
| **psycopg3** | Modern PostgreSQL adapter with binary protocol |
| **ConnectorX** | Zero-copy data loading into DataFrames |
| **Polars** | High-performance DataFrame library |

## Core Philosophy

1. **Database for storage, Polars for logic** - Fetch raw data, implement business logic in Polars
2. **Bulk operations over row-by-row** - Use COPY protocol for 10-100x performance
3. **Safe SQL composition** - Always use `psycopg.sql` module for dynamic SQL

## Quick Reference

### Reading Data

```python
# Time-series data with date range
df = client.read_table("public", "Table", start, end, ts_column="trade_day")

# Custom SQL (ConnectorX, high performance)
df = client.query('SELECT * FROM public."Table" WHERE active = true')
```

### Writing Data

```python
# Bulk insert (~10-100x faster than INSERT)
client.copy_from_df("public", "Table", df)

# Upsert (INSERT ON CONFLICT)
client.upsert_df("public", "Table", df, conflict_columns=["id", "date"])

# Replace time range (atomic DELETE + INSERT)
client.replace_df("public", "Table", start, end, df, ts_column="trade_day")
```

### SQL Safety

```python
from psycopg import sql

# ALWAYS use sql module for dynamic identifiers
query = sql.SQL("SELECT * FROM {schema}.{table}").format(
    schema=sql.Identifier("public"),
    table=sql.Identifier("MyTable"),
)
```

## Critical Rules

### ⚠️ NUMERIC Columns

```python
# CRITICAL: Use binary=False for NUMERIC columns
client.copy_from_df("public", "Table", df, binary=False)
client.upsert_df("public", "Table", df, conflict_columns=["id"], binary=False)
```

### ❌ Anti-Patterns

```python
# NEVER: String concatenation (SQL injection risk)
df = client.query(f"SELECT * FROM {user_input}")

# NEVER: Row-by-row operations (extremely slow)
for row in data:
    client.insert("public", "Table", [row])

# AVOID: Complex business logic in SQL
# Instead: Fetch raw data, process in Polars
```

## References

- **Detailed API Guide**: See [references/postgres.md](references/postgres.md)
