<overview>
Common PostgreSQL query patterns for sqlc. Includes CRUD, pagination, soft deletes, upserts, and advanced patterns like CTEs and dynamic filtering.
</overview>

<crud>
## Basic CRUD

```sql
-- name: CreateUser :one
INSERT INTO users (email, name)
VALUES (sqlc.arg(email), sqlc.arg(name))
RETURNING *;

-- name: GetUser :one
SELECT id, email, name, created_at, updated_at
FROM users
WHERE id = sqlc.arg(id);

-- name: UpdateUser :one
UPDATE users
SET name = sqlc.arg(name), updated_at = NOW()
WHERE id = sqlc.arg(id)
RETURNING *;

-- name: DeleteUser :exec
DELETE FROM users WHERE id = sqlc.arg(id);
```
</crud>

<pagination>
## Pagination

### Offset-Based (Simple)

```sql
-- name: ListUsersPaginated :many
SELECT id, email, name, created_at
FROM users
WHERE deleted_at IS NULL
ORDER BY created_at DESC
LIMIT sqlc.arg(page_size)
OFFSET sqlc.arg(page_offset);

-- name: CountUsers :one
SELECT COUNT(*) FROM users WHERE deleted_at IS NULL;
```

### Cursor-Based (Efficient for Large Datasets)

```sql
-- name: ListUsersAfterCursor :many
SELECT id, email, name, created_at
FROM users
WHERE deleted_at IS NULL
  AND (created_at, id) < (sqlc.arg(cursor_time), sqlc.arg(cursor_id))
ORDER BY created_at DESC, id DESC
LIMIT sqlc.arg(page_size);
```

**Cursor-based is faster:** No offset scanning for deep pages.
</pagination>

<soft_deletes>
## Soft Deletes

```sql
-- name: SoftDeleteUser :exec
UPDATE users SET deleted_at = NOW() WHERE id = sqlc.arg(id);

-- name: RestoreUser :exec
UPDATE users SET deleted_at = NULL WHERE id = sqlc.arg(id);

-- name: ListActiveUsers :many
SELECT id, email, name FROM users WHERE deleted_at IS NULL;

-- name: ListDeletedUsers :many
SELECT id, email, name, deleted_at FROM users WHERE deleted_at IS NOT NULL;

-- name: HardDeleteUser :exec
DELETE FROM users WHERE id = sqlc.arg(id) AND deleted_at IS NOT NULL;
```
</soft_deletes>

<upsert>
## Upsert (Insert or Update)

```sql
-- name: UpsertUser :one
INSERT INTO users (email, name)
VALUES (sqlc.arg(email), sqlc.arg(name))
ON CONFLICT (email) DO UPDATE SET
  name = EXCLUDED.name,
  updated_at = NOW()
RETURNING *;

-- Upsert with specific conflict target
-- name: UpsertUserByExternalID :one
INSERT INTO users (external_id, email, name)
VALUES (sqlc.arg(external_id), sqlc.arg(email), sqlc.arg(name))
ON CONFLICT (external_id) DO UPDATE SET
  email = EXCLUDED.email,
  name = EXCLUDED.name,
  updated_at = NOW()
RETURNING *;

-- Upsert doing nothing on conflict
-- name: InsertUserIfNotExists :one
INSERT INTO users (email, name)
VALUES (sqlc.arg(email), sqlc.arg(name))
ON CONFLICT (email) DO NOTHING
RETURNING *;
```
</upsert>

<optimistic_locking>
## Optimistic Locking

```sql
-- Add version column to schema:
-- ALTER TABLE users ADD COLUMN version INT NOT NULL DEFAULT 1;

-- name: UpdateUserWithVersion :execrows
UPDATE users
SET
  name = sqlc.arg(name),
  version = version + 1,
  updated_at = NOW()
WHERE id = sqlc.arg(id) AND version = sqlc.arg(expected_version);
```

**Go usage:**
```go
rowsAffected, err := q.UpdateUserWithVersion(ctx, UpdateUserWithVersionParams{
    ID:              userID,
    Name:            newName,
    ExpectedVersion: currentVersion,
})
if rowsAffected == 0 {
    return errors.New("concurrent modification detected")
}
```
</optimistic_locking>

<dynamic_filtering>
## Dynamic Filtering

```sql
-- name: SearchUsers :many
SELECT id, email, name, created_at
FROM users
WHERE
  deleted_at IS NULL
  AND (sqlc.narg(email_filter)::text IS NULL OR email ILIKE sqlc.narg(email_filter))
  AND (sqlc.narg(name_filter)::text IS NULL OR name ILIKE sqlc.narg(name_filter))
  AND (sqlc.narg(role_filter)::user_role IS NULL OR role = sqlc.narg(role_filter))
  AND (sqlc.narg(created_after)::timestamptz IS NULL OR created_at >= sqlc.narg(created_after))
ORDER BY created_at DESC
LIMIT sqlc.arg(limit);
```

**Pattern:** `(param IS NULL OR column = param)` - filter only when param provided.
</dynamic_filtering>

<joins>
## JOINs

### Simple JOIN

```sql
-- name: GetUserWithOrganization :one
SELECT
  u.id,
  u.email,
  u.name,
  o.id as organization_id,
  o.name as organization_name
FROM users u
JOIN organizations o ON u.organization_id = o.id
WHERE u.id = sqlc.arg(user_id);
```

### LEFT JOIN with NULLs

```sql
-- name: GetUserWithOptionalOrg :one
SELECT
  u.id,
  u.email,
  u.name,
  o.id as organization_id,
  o.name as organization_name
FROM users u
LEFT JOIN organizations o ON u.organization_id = o.id
WHERE u.id = sqlc.arg(user_id);
```

### Many-to-Many

```sql
-- name: GetUserRoles :many
SELECT r.id, r.name
FROM roles r
JOIN user_roles ur ON r.id = ur.role_id
WHERE ur.user_id = sqlc.arg(user_id);
```
</joins>

<cte>
## CTEs (Common Table Expressions)

### Simple CTE

```sql
-- name: GetUserWithStats :one
WITH user_orders AS (
  SELECT
    user_id,
    COUNT(*) as order_count,
    COALESCE(SUM(total), 0) as total_spent
  FROM orders
  WHERE user_id = sqlc.arg(user_id)
  GROUP BY user_id
)
SELECT
  u.id,
  u.email,
  u.name,
  COALESCE(uo.order_count, 0)::int as order_count,
  COALESCE(uo.total_spent, 0)::numeric as total_spent
FROM users u
LEFT JOIN user_orders uo ON u.id = uo.user_id
WHERE u.id = sqlc.arg(user_id);
```

### Recursive CTE (Hierarchies)

```sql
-- name: GetCategoryWithAncestors :many
WITH RECURSIVE ancestors AS (
  SELECT id, name, parent_id, 0 as depth
  FROM categories
  WHERE id = sqlc.arg(category_id)

  UNION ALL

  SELECT c.id, c.name, c.parent_id, a.depth + 1
  FROM categories c
  JOIN ancestors a ON c.id = a.parent_id
)
SELECT id, name, depth FROM ancestors ORDER BY depth DESC;
```
</cte>

<arrays>
## Array Operations

### IN Query with Array

```sql
-- name: GetUsersByIDs :many
SELECT id, email, name
FROM users
WHERE id = ANY(sqlc.arg(ids)::uuid[]);

-- name: GetUsersByEmails :many
SELECT id, email, name
FROM users
WHERE email = ANY(sqlc.arg(emails)::text[]);
```

### Array Column Operations

```sql
-- name: AddUserTag :exec
UPDATE users
SET tags = array_append(tags, sqlc.arg(tag))
WHERE id = sqlc.arg(id);

-- name: RemoveUserTag :exec
UPDATE users
SET tags = array_remove(tags, sqlc.arg(tag))
WHERE id = sqlc.arg(id);

-- name: GetUsersWithTag :many
SELECT id, email, name
FROM users
WHERE sqlc.arg(tag) = ANY(tags);
```
</arrays>

<aggregations>
## Aggregations

```sql
-- name: GetUserStats :one
SELECT
  COUNT(*) as total_users,
  COUNT(*) FILTER (WHERE role = 'admin') as admin_count,
  COUNT(*) FILTER (WHERE created_at > NOW() - INTERVAL '30 days') as new_users,
  MIN(created_at) as first_user_at,
  MAX(created_at) as last_user_at
FROM users
WHERE deleted_at IS NULL;

-- name: GetOrdersByMonth :many
SELECT
  DATE_TRUNC('month', created_at) as month,
  COUNT(*) as order_count,
  SUM(total) as total_revenue
FROM orders
WHERE created_at >= sqlc.arg(since)
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;
```
</aggregations>

<transactions>
## Transactions

sqlc generates a `WithTx` method for transaction support:

```go
tx, err := pool.Begin(ctx)
if err != nil {
    return err
}
defer tx.Rollback(ctx) // Safe even after commit

qtx := queries.WithTx(tx)

// All queries in transaction
user, err := qtx.CreateUser(ctx, CreateUserParams{...})
if err != nil {
    return err
}

_, err = qtx.CreateAuditLog(ctx, CreateAuditLogParams{...})
if err != nil {
    return err
}

return tx.Commit(ctx)
```
</transactions>

<null_handling>
## NULL Handling

### COALESCE for Defaults

```sql
-- name: GetUserDisplayName :one
SELECT COALESCE(display_name, email) as display_name
FROM users
WHERE id = sqlc.arg(id);

-- name: GetUserStats :one
SELECT
  COALESCE(order_count, 0) as order_count,
  COALESCE(total_spent, 0) as total_spent
FROM user_stats
WHERE user_id = sqlc.arg(user_id);
```

### NULLIF

```sql
-- name: DivideMetrics :one
SELECT
  total_orders,
  total_revenue,
  -- Avoid division by zero
  total_revenue / NULLIF(total_orders, 0) as avg_order_value
FROM daily_metrics
WHERE date = sqlc.arg(date);
```
</null_handling>
