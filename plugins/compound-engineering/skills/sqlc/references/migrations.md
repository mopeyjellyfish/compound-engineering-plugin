<overview>
PostgreSQL migration best practices for sqlc projects. Covers naming, structure, rollback safety, and the critical decision of when to edit vs create new migrations.
</overview>

<naming>
## Migration File Naming

Use sequential numbering with descriptive names:

```
sql/migrations/
├── 0001_create_users.sql
├── 0002_add_user_email_index.sql
├── 0003_create_organizations.sql
├── 0004_add_user_organization_fk.sql
└── 0005_add_user_roles.sql
```

**Rules:**
- 4-digit zero-padded number
- Underscore separator
- Snake_case description
- `.sql` extension
- One logical change per file
</naming>

<structure>
## Migration Structure

Use goose annotations for up/down migrations:

```sql
-- 0003_create_organizations.sql

-- +goose Up
CREATE TABLE IF NOT EXISTS organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_organizations_slug ON organizations(slug);

-- +goose Down
DROP INDEX IF EXISTS idx_organizations_slug;
DROP TABLE IF EXISTS organizations;
```

**Key principles:**
- Use `IF EXISTS` / `IF NOT EXISTS` for idempotency
- Include both Up and Down sections
- Down should reverse Up completely
- Order matters: create before reference, drop in reverse
</structure>

<decision_workflow>
## Edit vs Create New Migration

**CRITICAL DECISION TREE:**

```
Is this migration merged to main/master?
├── YES → ALWAYS create NEW migration
│         (Never edit merged migrations)
│
└── NO → Is the app deployed to production?
    ├── YES → ALWAYS create NEW migration
    │         (Production may have run it)
    │
    └── NO → ASK USER:
            "Migration exists but isn't merged. Options:
            1. Edit existing (simpler, requires DB reset)
            2. Create new (safer, preserves history)"
```

**After editing existing migration:**
```bash
# Must reset local database
goose -dir sql/migrations postgres "$DATABASE_URL" reset
goose -dir sql/migrations postgres "$DATABASE_URL" up
sqlc generate
```

**Questions to ask user:**
1. "Is this migration already merged to main?"
2. "Has this app been deployed to production?"
3. "Do you want to edit the existing migration or create a new one?"
</decision_workflow>

<column_operations>
## Column Operations

### Adding Columns

```sql
-- +goose Up
-- NOT NULL columns need defaults
ALTER TABLE users ADD COLUMN role user_role NOT NULL DEFAULT 'member';

-- Nullable columns are simpler
ALTER TABLE users ADD COLUMN bio TEXT;

-- +goose Down
ALTER TABLE users DROP COLUMN IF EXISTS role;
ALTER TABLE users DROP COLUMN IF EXISTS bio;
```

### Renaming Columns (Safe)

```sql
-- +goose Up
ALTER TABLE users RENAME COLUMN legacy_name TO display_name;

-- +goose Down
ALTER TABLE users RENAME COLUMN display_name TO legacy_name;
```

### Dropping Columns (Dangerous)

```sql
-- DANGEROUS: Data loss on rollback!
-- Consider renaming first, dropping later

-- +goose Up
-- Step 1: Rename to deprecated (in one migration)
ALTER TABLE users RENAME COLUMN old_field TO old_field_deprecated;

-- Step 2: Drop after confirming no usage (separate migration, later)
ALTER TABLE users DROP COLUMN old_field_deprecated;
```
</column_operations>

<indexes>
## Index Operations

### Create Index (Production-Safe)

```sql
-- +goose Up
-- +goose StatementBegin
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email ON users(email);
-- +goose StatementEnd

-- +goose Down
DROP INDEX CONCURRENTLY IF EXISTS idx_users_email;
```

**CONCURRENTLY:** Doesn't lock the table. Required for production.
**StatementBegin/End:** Required for CONCURRENTLY in goose.

### Unique Index

```sql
-- +goose Up
-- +goose StatementBegin
CREATE UNIQUE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email_unique ON users(email);
-- +goose StatementEnd
```

### Partial Index

```sql
-- +goose Up
-- +goose StatementBegin
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_active_users
ON users(created_at) WHERE deleted_at IS NULL;
-- +goose StatementEnd
```
</indexes>

<foreign_keys>
## Foreign Key Operations

```sql
-- +goose Up
ALTER TABLE users
  ADD COLUMN organization_id UUID,
  ADD CONSTRAINT fk_users_organization
    FOREIGN KEY (organization_id)
    REFERENCES organizations(id)
    ON DELETE SET NULL;

-- Always index foreign keys
CREATE INDEX IF NOT EXISTS idx_users_organization_id ON users(organization_id);

-- +goose Down
ALTER TABLE users DROP CONSTRAINT IF EXISTS fk_users_organization;
DROP INDEX IF EXISTS idx_users_organization_id;
ALTER TABLE users DROP COLUMN IF EXISTS organization_id;
```

**ON DELETE options:**
- `CASCADE` - Delete children when parent deleted
- `SET NULL` - Set FK to NULL when parent deleted
- `RESTRICT` - Prevent parent deletion if children exist
- `NO ACTION` - Same as RESTRICT (default)
</foreign_keys>

<enums>
## Enum Operations

### Create Enum

```sql
-- +goose Up
CREATE TYPE user_role AS ENUM ('admin', 'member', 'guest');

-- +goose Down
DROP TYPE IF EXISTS user_role;
```

### Add Enum Value

```sql
-- +goose Up
ALTER TYPE user_role ADD VALUE 'moderator';

-- +goose Down
-- Cannot remove enum values in PostgreSQL!
-- Document this limitation
```

### Safe Enum Addition (Idempotent)

```sql
-- +goose Up
DO $$ BEGIN
  IF NOT EXISTS (SELECT 1 FROM pg_type WHERE typname = 'user_role') THEN
    CREATE TYPE user_role AS ENUM ('admin', 'member', 'guest');
  END IF;
END $$;
```
</enums>

<rollback_safety>
## Rollback Safety Checklist

Before writing a migration, consider:

1. **Can this be reversed?**
   - Column drops lose data
   - Type changes may fail if data doesn't fit
   - Enum value removal is impossible in PostgreSQL

2. **Is it idempotent?**
   - Use `IF EXISTS` / `IF NOT EXISTS`
   - Handle partial failures

3. **Will it lock tables?**
   - `CREATE INDEX CONCURRENTLY` for large tables
   - Consider off-hours for schema changes

4. **What happens on failure?**
   - Transaction rollback?
   - Partial state cleanup?

**Template for risky migrations:**
```sql
-- +goose Up
-- WARNING: This migration cannot be fully reversed.
-- Data in [column_name] will be lost on rollback.
-- Backup recommended before running.

ALTER TABLE users DROP COLUMN legacy_data;

-- +goose Down
-- NOTE: Cannot restore data. Only recreates column structure.
ALTER TABLE users ADD COLUMN legacy_data JSONB;
```
</rollback_safety>
