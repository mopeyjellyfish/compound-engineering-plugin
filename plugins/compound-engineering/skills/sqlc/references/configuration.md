<overview>
SQLc configuration reference for sqlc.yaml. Covers recommended settings, type overrides, and common configurations for Go + PostgreSQL projects.
</overview>

<recommended_config>
## Recommended sqlc.yaml

```yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "sql/queries"
    schema: "sql/migrations"
    gen:
      go:
        package: "db"
        out: "internal/db"
        sql_package: "pgx/v5"
        emit_json_tags: true
        emit_prepared_queries: false
        emit_interface: true
        emit_exact_table_names: false
        emit_empty_slices: true
        emit_exported_queries: false
        emit_result_struct_pointers: false
        emit_params_struct_pointers: false
        emit_methods_with_db_argument: false
        emit_pointers_for_null_types: true
        emit_enum_valid_method: true
        emit_all_enum_values: true
        json_tags_case_style: "snake"
        output_db_file_name: "db.go"
        output_models_file_name: "models.go"
        output_querier_file_name: "querier.go"
```
</recommended_config>

<key_options>
## Key Configuration Options

| Option | Recommended | Why |
|--------|-------------|-----|
| `sql_package` | `pgx/v5` | Best PostgreSQL driver, connection pooling |
| `emit_json_tags` | `true` | Enables JSON serialization for APIs |
| `emit_empty_slices` | `true` | Returns `[]` not `nil` for empty :many |
| `emit_pointers_for_null_types` | `true` | `*string` instead of `sql.NullString` |
| `emit_enum_valid_method` | `true` | Adds `Valid()` method to enums |
| `emit_interface` | `true` | Generates `Querier` interface for mocking |
| `emit_prepared_queries` | `false` | pgx handles this automatically |

### sql_package Options

```yaml
sql_package: "pgx/v5"      # Recommended for PostgreSQL
sql_package: "database/sql" # Standard library (less features)
```

### json_tags_case_style Options

```yaml
json_tags_case_style: "snake"  # user_name (recommended for APIs)
json_tags_case_style: "camel"  # userName
json_tags_case_style: "pascal" # UserName
json_tags_case_style: "none"   # No JSON tags
```
</key_options>

<type_overrides>
## Type Overrides

Override default type mappings for specific columns or database types:

```yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "sql/queries"
    schema: "sql/migrations"
    gen:
      go:
        package: "db"
        out: "internal/db"
        overrides:
          # Override specific columns
          - column: "users.id"
            go_type: "github.com/google/uuid.UUID"
          - column: "users.metadata"
            go_type: "json.RawMessage"

          # Override by database type (applies globally)
          - db_type: "uuid"
            go_type: "github.com/google/uuid.UUID"
          - db_type: "timestamptz"
            go_type: "time.Time"
          - db_type: "jsonb"
            go_type: "json.RawMessage"
          - db_type: "inet"
            go_type: "netip.Addr"
```

### Common Type Overrides

| PostgreSQL Type | Go Type | Import |
|----------------|---------|--------|
| `uuid` | `uuid.UUID` | `github.com/google/uuid` |
| `timestamptz` | `time.Time` | (built-in) |
| `jsonb` | `json.RawMessage` | `encoding/json` |
| `inet` | `netip.Addr` | `net/netip` |
| `cidr` | `netip.Prefix` | `net/netip` |
| `numeric` | `decimal.Decimal` | `github.com/shopspring/decimal` |

### Nullable Type Overrides

```yaml
overrides:
  - db_type: "uuid"
    nullable: true
    go_type:
      import: "github.com/google/uuid"
      type: "NullUUID"
```
</type_overrides>

<multiple_schemas>
## Multiple Schemas/Databases

```yaml
version: "2"
sql:
  # Main database
  - engine: "postgresql"
    queries: "sql/queries/main"
    schema: "sql/migrations/main"
    gen:
      go:
        package: "maindb"
        out: "internal/db/main"

  # Analytics database
  - engine: "postgresql"
    queries: "sql/queries/analytics"
    schema: "sql/migrations/analytics"
    gen:
      go:
        package: "analyticsdb"
        out: "internal/db/analytics"
```
</multiple_schemas>

<directory_structure>
## Recommended Directory Structure

```
project/
├── sql/
│   ├── migrations/
│   │   ├── 0001_create_users.sql
│   │   ├── 0002_create_organizations.sql
│   │   └── ...
│   ├── queries/
│   │   ├── users.sql
│   │   ├── organizations.sql
│   │   └── ...
│   └── schema.sql          # Optional: full schema dump
├── internal/
│   └── db/
│       ├── db.go           # Generated: connection handling
│       ├── models.go       # Generated: struct definitions
│       ├── querier.go      # Generated: interface
│       ├── users.sql.go    # Generated: query methods
│       └── ...
└── sqlc.yaml
```

### Query File Organization

One file per table/domain:
```
sql/queries/
├── users.sql          # User CRUD
├── organizations.sql  # Organization CRUD
├── memberships.sql    # User-Org relationships
└── audit_logs.sql     # Audit logging
```
</directory_structure>

<strict_mode>
## Strict Mode

Enable strict mode to catch more errors at compile time:

```yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "sql/queries"
    schema: "sql/migrations"
    strict_function_checks: true
    gen:
      go:
        package: "db"
        out: "internal/db"
```

**What strict mode catches:**
- Unused parameters in queries
- Missing return columns
- Type mismatches
</strict_mode>

<database_connection>
## Database Connection (pgx/v5)

```go
package main

import (
    "context"
    "github.com/jackc/pgx/v5/pgxpool"
    "yourproject/internal/db"
)

func main() {
    ctx := context.Background()

    // Connection pool (recommended)
    pool, err := pgxpool.New(ctx, os.Getenv("DATABASE_URL"))
    if err != nil {
        log.Fatal(err)
    }
    defer pool.Close()

    // Create queries instance
    queries := db.New(pool)

    // Use queries
    user, err := queries.GetUser(ctx, userID)
}
```
</database_connection>
