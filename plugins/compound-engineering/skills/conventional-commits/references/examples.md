<overview>
Full commit message examples for different scenarios.
</overview>

<simple_feature>
## Simple Feature

```bash
git commit -m "$(cat <<'EOF'
feat(api): add rate limiting to REST endpoints
EOF
)"
```
</simple_feature>

<bug_fix>
## Bug Fix with Context

```bash
git commit -m "$(cat <<'EOF'
fix(parser): handle malformed UTF-8 sequences

Previously crashed on invalid byte sequences. Now replaces
invalid sequences with Unicode replacement character.

Fixes #123
EOF
)"
```
</bug_fix>

<refactor>
## Refactor

```bash
git commit -m "$(cat <<'EOF'
refactor(db): extract connection pool management

Moved connection pooling logic to dedicated module
for better testability and reuse.
EOF
)"
```
</refactor>

<breaking_change>
## Breaking Change

**ONLY after user confirmation:**

```bash
git commit -m "$(cat <<'EOF'
feat(api)!: require authentication for all endpoints

BREAKING CHANGE: All API endpoints now require valid
JWT tokens. Anonymous access has been removed.
EOF
)"
```
</breaking_change>

<documentation>
## Documentation

```bash
git commit -m "$(cat <<'EOF'
docs(api): add authentication examples
EOF
)"
```
</documentation>

<performance>
## Performance

```bash
git commit -m "$(cat <<'EOF'
perf(queries): add database index for user lookups

Reduces lookup time from 200ms to 5ms for users table.
EOF
)"
```
</performance>

<chore>
## Chore/Maintenance

```bash
git commit -m "$(cat <<'EOF'
chore(deps): update dependencies to latest versions
EOF
)"
```
</chore>

<multi_scope>
## Multiple Scopes (Avoid)

Prefer separate commits for separate concerns:

```bash
# ❌ BAD - Too many things
git commit -m "feat: add auth and fix parser and update docs"

# ✅ GOOD - Separate commits
git commit -m "feat(auth): add OAuth2 support"
git commit -m "fix(parser): handle empty input"
git commit -m "docs(readme): update auth section"
```
</multi_scope>
