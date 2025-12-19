---
name: conventional-commits
description: Ensures commits follow conventional commits specification with proper pre-commit handling and semantic versioning alignment. Use when staging/committing code changes.
---

# Conventional Commits

All commits must follow the conventional commits specification for automated changelog generation and semantic versioning.

## Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Quick Type Reference

| Type | SemVer | When to Use |
|------|--------|-------------|
| `feat` | MINOR | New feature |
| `fix` | PATCH | Bug fix |
| `docs` | PATCH | Documentation |
| `refactor` | PATCH | Code restructure |
| `perf` | PATCH | Performance |
| `test` | PATCH | Tests |
| `chore` | PATCH | Maintenance |

**Full types:** `references/commit-types.md`

## Commit Workflow

### 1. Stage Changes
```bash
git add <files>
git diff --staged  # Verify
```

### 2. Determine Type
- New functionality → `feat`
- Bug fix → `fix`
- Docs → `docs`
- Restructure → `refactor`

### 3. Write Message (HEREDOC)
```bash
git commit -m "$(cat <<'EOF'
feat(scope): short description

Optional body explaining why.
EOF
)"
```

### 4. Handle Pre-Commit
**CRITICAL:** Never use `--no-verify`. Always fix issues.

If files auto-modified → stage and commit again
If hook fails → fix the issue, then commit

**Full guide:** `references/pre-commit-handling.md`

## Breaking Changes

**NEVER assume breaking. Always confirm with user first.**

Ask: "Is this a breaking change requiring MAJOR version bump?"

```bash
feat(api)!: change auth endpoint

BREAKING CHANGE: Now requires OAuth2 tokens
```

## Message Rules

**DO:**
- Specific descriptions
- Scope when applicable
- Body for "why" on complex changes

**DON'T:**
- Reference Claude/AI
- Vague words ("updates", "changes")
- Multiple unrelated changes (split commits)

## Quick Examples

```bash
# Feature
feat(auth): add JWT refresh endpoint

# Bug fix
fix(parser): handle null values in response

# Refactor
refactor(db): extract connection pooling

# Breaking (after user confirms)
feat(api)!: require auth for all endpoints
```

**More examples:** `references/examples.md`

## Reference Files

| File | Content |
|------|---------|
| `references/commit-types.md` | All types, scopes, breaking changes |
| `references/pre-commit-handling.md` | Hook error handling |
| `references/examples.md` | Full commit examples |
