---
name: db-schema-explorer
description: >
  Use this skill when the user wants to explore/discover database schemas, tables, columns,
  and data in any connected database. Trigger on phrases like "查看数据源", "发现schema",
  "discover schema", "列出表", "list tables", "查看表结构", "describe table", "浏览数据库",
  "browse databases", "查询数据", "query data", "explore database", "db-schema-explorer".
  This skill uses db_connect_mcp to inspect schemas, list tables, describe columns,
  sample data, search objects, and analyze queries. IMPORTANT: These tools are easy to
  confuse with other MCPs' tools (e.g. seatunnel-mcp-server*). Always verify you are
  calling the correct tool.
user-invocable: true
disable-model-invocation: false
---

# DB Schema Explorer

## MCP Server Info
- **Package**: `db-connect-mcp` v0.6.1
- **Toolkit**: 9 tools for MySQL (all except `analyze_column` — only PostgreSQL supports it)
- **Connection**: stdio transport, connects via configured `DATABASE_URL` env

## When to Use
- User wants to explore/browse any database schemas, tables, or columns
- User asks "list tables", "show schema of table Y", "describe table Z"
- User types `/db-schema-explorer` or similar discovery commands
- User wants to sample data from a table, search for database objects, or analyze table relationships
- User needs to execute ad-hoc queries or get query execution plans
- User wants to understand table structures before working with data pipelines

## ⚠️ CRITICAL: Disambiguation Warning

These tools are **easy to confuse** with tools from other MCP servers (e.g. `seatunnel-mcp-server`, `seatunnel-mcp-server-local`). Always verify you are calling the correct tool.
**DO NOT** fall back to calling other MCP tools — use only the tools listed below.

## Available Tools (9 for MySQL)

| Tool | VS Code Prefixed Name | Parameters | Purpose |
|------|----------------------|------------|---------|
| `get_database_info` | `mcp_db-connect-mc_get_database_info` | *(none)* | Get database info (version, size, capabilities) |
| `list_schemas` | `mcp_db-connect-mc_list_schemas` | *(none)* | List all schemas in the database |
| `list_tables` | `mcp_db-connect-mc_list_tables` | `schema` (str, optional), `include_views` (bool, default: true) | List all tables/views in a schema |
| `describe_table` | `mcp_db-connect-mc_describe_table` | `table` (str, **required**), `schema` (str, optional) | Get columns, indexes, constraints |
| `execute_query` | `mcp_db-connect-mc_execute_query` | `query` (str, **required**), `limit` (int, default: 1000) | Execute read-only SQL (SELECT/WITH/EXPLAIN) |
| `sample_data` | `mcp_db-connect-mc_sample_data` | `table` (str, **required**), `schema` (str, optional), `limit` (int, default: 100) | Sample rows from a table |
| `search_objects` | `mcp_db-connect-mc_search_objects` | `pattern` (str, **required**), `object_types` (array, optional), `detail_level` ("names"\|"summary"\|"full", default: "summary"), `schema` (str, optional), `table` (str, optional), `limit` (int, 1-1000, default: 100) | Search DB objects by LIKE pattern |
| `get_table_relationships` | `mcp_db-connect-mc_get_table_relationships` | `table` (str, **required**), `schema` (str, optional) | Get foreign key relationships |
| `explain_query` | `mcp_db-connect-mc_explain_query` | `query` (str, **required**), `analyze` (bool, default: false) | Get query execution plan |

> **Note**: The `mcp_db-connect-mc_` prefix is added by VS Code at runtime to avoid naming conflicts between MCP servers. You can call tools by either name — both resolve to the same MCP server.

## Workflow

### Step 1: Identify Discovery Goal
Ask what the user wants to do:
- **Get database info / list schemas** → Step 2a
- **List tables** → Step 2b
- **Describe table structure** → Step 2c
- **Sample / query data** → Step 2d
- **Search objects / relationships** → Step 2e

### Step 2a: Database Info & Schemas
```python
# Get database information (version, size, capabilities) — no parameters needed
get_database_info()

# List all schemas — no parameters needed
list_schemas()
```

### Step 2b: List Tables
```python
# List all tables in a specific schema
list_tables(schema="<schema_name>")

# Include views as well
list_tables(schema="<schema_name>", include_views=True)

# Omit schema to see all tables across schemas
list_tables()
```

### Step 2c: Describe Table Structure
```python
# Get comprehensive info: columns, indexes, constraints
describe_table(table="<table_name>", schema="<schema_name>")
```

### Step 2d: Sample & Query Data
```python
# Get a sample of rows from a table
sample_data(table="<table_name>", schema="<schema_name>", limit=10)

# Execute an ad-hoc read-only SQL query
execute_query(query="SELECT * FROM schema.table WHERE ...", limit=100)

# Get the query execution plan (MySQL supports this)
explain_query(query="SELECT * FROM schema.table WHERE ...", analyze=False)
```

### Step 2e: Search & Relationships
```python
# Search for database objects by name pattern (SQL LIKE syntax)
search_objects(pattern="%keyword%")
search_objects(pattern="%user%", object_types=["table", "view"])
search_objects(pattern="%order%", detail_level="full", limit=50)

# Get foreign key relationships for a table
get_table_relationships(table="<table_name>", schema="<schema_name>")
```

## Typical Discovery Sequence

A common end-to-end flow:

1. **Get database info** → `get_database_info()` — see version, capabilities
2. **List schemas** → `list_schemas()` — see all databases/schemas
3. **List tables** → `list_tables(schema="<name>")` — explore a schema
4. **Describe a table** → `describe_table(table="<name>", schema="<name>")` — see columns + indexes
5. **(Optional) Sample data** → `sample_data(table="<name>", schema="<name>", limit=5)`
6. **(Optional) Check relationships** → `get_table_relationships(table="<name>", schema="<name>")`

## Important Notes

- **Authentication**: Depends on `DATABASE_URL` configured in `mcp.json`
- **Do NOT confuse with other MCPs**: Always double-check tool names before calling
- **Read-only only**: `execute_query` only supports SELECT/WITH/EXPLAIN — no INSERT/UPDATE/DELETE
- **Search with LIKE patterns**: `search_objects` supports SQL wildcards: `%` for any chars, `_` for single char
- **`analyze_column` is unavailable**: MySQL's `advanced_stats = False`, so column statistics analysis is only available for PostgreSQL
