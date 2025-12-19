---
name: sqlc
description: PostgreSQL query and migration expert for sqlc. Use when working with `*/sql/*`, `*/sql/queries/*`, or `*/sql/migrations/*` directories. Ensures idiomatic, well-structured queries and migrations following sqlc best practices.
---

# SQLc PostgreSQL Expert

Expert guidance for writing idiomatic PostgreSQL queries and migrations using sqlc.

**Official Docs:** https://docs.sqlc.dev/en/latest/

## When This Skill Triggers

- `*/sql/queries/*.sql` - Query files
- `*/sql/migrations/*.sql` - Migration files
- `*/sql/*.sql` - Any SQL files
- `sqlc.yaml` / `sqlc.json` - Configuration

## Quick Reference

### Query Annotations

| Annotation | Returns | Use Case |
|-----------|---------|----------|
| `:one` | Single row | Get by ID/unique |
| `:many` | Slice | List/search |
| `:exec` | Error only | UPDATE/DELETE |
| `:execrows` | Row count | Bulk ops, optimistic locking |
| `:copyfrom` | Row count | Bulk insert (COPY) |
| `:batchone` | Batch results | Multi-fetch |

**Full reference:** `references/query-annotations.md`

### Parameter Syntax

```sql
$1, $2, $3           -- Positional (simple)
sqlc.arg(name)       -- Named (recommended)
sqlc.narg(name)      -- Nullable/optional
sqlc.slice(ids)      -- Array parameter
```

### Essential Rules

1. **Never use `SELECT *`** - Explicit column lists only
2. **Named parameters** - Use `sqlc.arg()` for clarity
3. **Handle NULLs** - Use `COALESCE` for defaults
4. **Index foreign keys** - Always create FK indexes

## Migration Decision Workflow

**CRITICAL:** Before editing migrations, determine app status.

```
Migration merged to main?
├── YES → Create NEW migration (never edit)
└── NO → App deployed to production?
    ├── YES → Create NEW migration
    └── NO → ASK USER: Edit existing or create new?
```

**Always ask:**
1. "Is this migration merged to main?"
2. "Has the app been deployed to production?"

**Full reference:** `references/migrations.md`

## Common Patterns

### UPSERT

```sql
-- name: UpsertUser :one
INSERT INTO users (email, name) VALUES ($1, $2)
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name
RETURNING *;
```

### Soft Delete

```sql
-- name: SoftDeleteUser :exec
UPDATE users SET deleted_at = NOW() WHERE id = $1;

-- name: ListActiveUsers :many
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Optimistic Locking

```sql
-- name: UpdateUserWithVersion :execrows
UPDATE users SET name = $2, version = version + 1
WHERE id = $1 AND version = $3;
-- Check: if rows == 0, concurrent modification
```

### Dynamic Filtering

```sql
-- name: SearchUsers :many
SELECT * FROM users
WHERE (sqlc.narg(email)::text IS NULL OR email ILIKE sqlc.narg(email))
  AND (sqlc.narg(role)::user_role IS NULL OR role = sqlc.narg(role))
LIMIT sqlc.arg(limit);
```

**Full reference:** `references/patterns.md`

## Configuration

Recommended `sqlc.yaml`:

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
        emit_empty_slices: true
        emit_pointers_for_null_types: true
        emit_enum_valid_method: true
```

**Full reference:** `references/configuration.md`

## Reference Files

For detailed guidance, read these reference files:

| File | Content |
|------|---------|
| `references/query-annotations.md` | All annotation types, parameters, casting |
| `references/migrations.md` | Migration structure, edit vs new decision, rollback |
| `references/configuration.md` | sqlc.yaml options, type overrides |
| `references/patterns.md` | CRUD, pagination, CTEs, arrays, transactions |

## Fetch More Details

For topics not covered here:
```
WebFetch: https://docs.sqlc.dev/en/latest/
```

Topics: configuration, queries, migrations, type overrides, plugins, joins, embedding
