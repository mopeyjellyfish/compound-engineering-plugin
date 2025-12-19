<overview>
SQLc query annotation reference. Every query MUST have a name and return type annotation.
Format: `-- name: QueryName :returntype`
</overview>

<annotation name="one">
## :one - Single Row

Returns exactly one row. Errors if no rows found.

```sql
-- name: GetUser :one
SELECT id, email, name, created_at FROM users WHERE id = $1;

-- name: GetUserByEmail :one
SELECT id, email, name FROM users WHERE email = $1;
```

**Go signature:** `func (q *Queries) GetUser(ctx context.Context, id uuid.UUID) (User, error)`

**Use when:** Fetching by primary key or unique constraint.
</annotation>

<annotation name="many">
## :many - Multiple Rows

Returns slice of rows. Empty slice if no matches (not error).

```sql
-- name: ListUsers :many
SELECT id, email, name, created_at FROM users ORDER BY created_at DESC;

-- name: ListUsersByOrg :many
SELECT id, email, name FROM users WHERE organization_id = $1;
```

**Go signature:** `func (q *Queries) ListUsers(ctx context.Context) ([]User, error)`

**Use when:** Listing, searching, filtering.
</annotation>

<annotation name="exec">
## :exec - No Return Value

For UPDATE/DELETE when you don't need affected count.

```sql
-- name: UpdateUserName :exec
UPDATE users SET name = $2, updated_at = NOW() WHERE id = $1;

-- name: DeleteUser :exec
DELETE FROM users WHERE id = $1;
```

**Go signature:** `func (q *Queries) UpdateUserName(ctx context.Context, arg UpdateUserNameParams) error`

**Use when:** Simple updates/deletes where success is enough.
</annotation>

<annotation name="execrows">
## :execrows - Affected Row Count

Returns number of rows affected.

```sql
-- name: SoftDeleteInactiveUsers :execrows
UPDATE users SET deleted_at = NOW() WHERE last_active_at < $1 AND deleted_at IS NULL;

-- name: UpdateUserWithVersion :execrows
UPDATE users SET name = $2, version = version + 1 WHERE id = $1 AND version = $3;
```

**Go signature:** `func (q *Queries) SoftDeleteInactiveUsers(ctx context.Context, before time.Time) (int64, error)`

**Use when:** Optimistic locking, bulk operations, need to verify rows changed.
</annotation>

<annotation name="execresult">
## :execresult - Full sql.Result

Returns complete sql.Result for LastInsertId() and RowsAffected().

```sql
-- name: InsertUser :execresult
INSERT INTO users (email, name) VALUES ($1, $2);
```

**Go signature:** `func (q *Queries) InsertUser(ctx context.Context, arg InsertUserParams) (sql.Result, error)`

**Use when:** Need both last insert ID and rows affected (rare with PostgreSQL).
</annotation>

<annotation name="copyfrom">
## :copyfrom - Bulk Insert via COPY

Uses PostgreSQL COPY for fastest bulk inserts.

```sql
-- name: BulkInsertUsers :copyfrom
INSERT INTO users (email, name, created_at) VALUES ($1, $2, $3);
```

**Go signature:** `func (q *Queries) BulkInsertUsers(ctx context.Context, arg []BulkInsertUsersParams) (int64, error)`

**Use when:** Inserting thousands of rows. 10-100x faster than individual inserts.
</annotation>

<annotation name="batch">
## :batchone / :batchmany / :batchexec - Batch Operations

Execute multiple queries in a single round-trip.

```sql
-- name: GetUserBatch :batchone
SELECT id, email, name FROM users WHERE id = $1;

-- name: CreateUserBatch :batchone
INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *;

-- name: DeleteUserBatch :batchexec
DELETE FROM users WHERE id = $1;
```

**Go usage:**
```go
batch := q.GetUserBatch(ctx, []uuid.UUID{id1, id2, id3})
defer batch.Close()
for batch.Next() {
    user, err := batch.Value()
}
```

**Use when:** Need to fetch/modify multiple records efficiently.
</annotation>

<parameters>
## Parameter Syntax

### Positional Parameters
```sql
$1, $2, $3  -- Simple, but unclear with many params
```

### Named Parameters (Recommended)
```sql
sqlc.arg(user_id)    -- Required parameter
sqlc.narg(name)      -- Nullable/optional parameter
sqlc.slice(ids)      -- Array/slice parameter
```

### Examples
```sql
-- Named parameters for clarity
-- name: CreateUser :one
INSERT INTO users (email, name, role)
VALUES (sqlc.arg(email), sqlc.arg(name), sqlc.arg(role))
RETURNING *;

-- Nullable parameter
-- name: UpdateUserName :exec
UPDATE users SET name = sqlc.narg(name) WHERE id = sqlc.arg(id);

-- Array parameter
-- name: GetUsersByIDs :many
SELECT * FROM users WHERE id = ANY(sqlc.arg(ids)::uuid[]);
```
</parameters>

<type_casting>
## Type Casting

Cast parameters to PostgreSQL types:

```sql
$1::uuid           -- UUID type
$1::user_role      -- Enum type
$1::text[]         -- Text array
$1::jsonb          -- JSONB
$1::timestamptz    -- Timestamp with timezone
```

Example with enum:
```sql
-- name: CreateUser :one
INSERT INTO users (email, role) VALUES ($1, $2::user_role) RETURNING *;
```
</type_casting>
