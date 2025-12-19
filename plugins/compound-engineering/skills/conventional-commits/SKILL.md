---
name: conventional-commits
description: This skill ensures all commits follow conventional commits specification with proper pre-commit handling and semantic versioning alignment. Use when staging/committing code changes.
---

# Conventional Commits

Follow this skill when creating git commits. All commits must follow the conventional commits specification for automated changelog generation and semantic versioning.

## Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Types

| Type | SemVer | Description |
|------|--------|-------------|
| `feat` | MINOR | New feature or capability |
| `fix` | PATCH | Bug fix |
| `docs` | PATCH | Documentation only changes |
| `style` | PATCH | Formatting, whitespace (no code change) |
| `refactor` | PATCH | Code change that neither fixes bug nor adds feature |
| `perf` | PATCH | Performance improvement |
| `test` | PATCH | Adding or correcting tests |
| `build` | PATCH | Build system or external dependencies |
| `ci` | PATCH | CI configuration changes |
| `chore` | PATCH | Other changes that don't modify src or test files |

## Breaking Changes

**CRITICAL: Never assume a change is breaking. Always confirm with user first.**

Breaking changes require MAJOR version bump and MUST be confirmed:

```bash
# Option 1: Using ! after type
feat(api)!: change authentication endpoint

# Option 2: Footer notation
feat(api): change authentication endpoint

BREAKING CHANGE: The /auth endpoint now requires OAuth2 tokens instead of API keys
```

Before marking anything as breaking, ask the user:
> "This change affects [X]. Is this a breaking change that requires a MAJOR version bump?"

## Scope Guidelines

Use scope to clarify what's changed:

```
feat(auth): add OAuth2 support
fix(parser): handle empty input
docs(readme): update installation steps
refactor(db): extract connection pooling
```

Common scopes: `api`, `auth`, `db`, `ui`, `cli`, `config`, `deps`

## Commit Workflow

### Step 1: Stage Changes

```bash
git add <files>
git status  # Review staged changes
git diff --staged  # Verify changes are correct
```

### Step 2: Analyze Changes for Type

Determine the appropriate type:
- Adding new functionality → `feat`
- Fixing a bug → `fix`
- Documentation updates → `docs`
- Restructuring code → `refactor`
- Performance improvements → `perf`
- Adding tests → `test`

### Step 3: Write Commit Message

Write clear, specific descriptions:

```bash
# ✅ GOOD - Specific, describes what changed
feat(auth): add JWT token refresh endpoint

# ❌ BAD - Vague, doesn't explain the change
feat: update auth

# ✅ GOOD - Clear scope and action
fix(parser): handle null values in JSON response

# ❌ BAD - No context
fix: bug fix
```

### Step 4: Commit with HEREDOC

Always use HEREDOC for multi-line commits:

```bash
git commit -m "$(cat <<'EOF'
feat(scope): short description

Optional body explaining the motivation and context.
What changed and why, not how.
EOF
)"
```

### Step 5: Handle Pre-Commit Hooks

**CRITICAL: Never ignore, bypass, or work around pre-commit issues.**

After running `git commit`:

1. **Read the pre-commit output completely**
2. **Check for any auto-modifications** (formatting, linting fixes)
3. **If files were modified**, stage them and commit again:
   ```bash
   git add .
   git commit -m "$(cat <<'EOF'
   style: apply auto-formatting
   EOF
   )"
   ```
4. **If pre-commit fails**, fix the underlying issue:
   ```bash
   # Read the error message
   # Fix the actual problem
   # Stage the fix
   # Commit again with original message
   ```

**Never use:**
- `--no-verify` to skip hooks
- `--no-gpg-sign` unless explicitly required
- Workarounds that "trick" pre-commit into passing

### Step 6: Verify Commit

```bash
git log -1 --oneline  # Verify commit message
git status  # Ensure clean working tree
```

## Message Requirements

### DO include:
- Specific changes made
- Scope when applicable
- Body explaining "why" for complex changes
- Footer with issue references if applicable

### DO NOT include:
- References to Claude, AI, or being generated
- Vague descriptions like "updates" or "changes"
- Multiple unrelated changes (split into separate commits)
- The "how" in the description (save for body if needed)

## Examples

### Simple Feature

```bash
git commit -m "$(cat <<'EOF'
feat(api): add rate limiting to REST endpoints
EOF
)"
```

### Bug Fix with Context

```bash
git commit -m "$(cat <<'EOF'
fix(parser): handle malformed UTF-8 sequences

Previously crashed on invalid byte sequences. Now replaces
invalid sequences with Unicode replacement character.

Fixes #123
EOF
)"
```

### Refactor

```bash
git commit -m "$(cat <<'EOF'
refactor(db): extract connection pool management

Moved connection pooling logic to dedicated module
for better testability and reuse.
EOF
)"
```

### Breaking Change (ONLY after user confirmation)

```bash
git commit -m "$(cat <<'EOF'
feat(api)!: require authentication for all endpoints

BREAKING CHANGE: All API endpoints now require valid
JWT tokens. Anonymous access has been removed.
EOF
)"
```

### Documentation

```bash
git commit -m "$(cat <<'EOF'
docs(api): add authentication examples
EOF
)"
```

## Pre-Commit Error Handling

### Linting Errors

```
✗ eslint: Found 3 errors
```

**Fix it:**
1. Read the specific errors
2. Fix each error in the source code
3. Stage the fixes: `git add .`
4. Commit again with the same message

### Formatting Changes

```
✓ prettier: Fixed 2 files
```

**Handle it:**
1. Pre-commit auto-fixed files
2. Stage the changes: `git add .`
3. Commit again (same message, or `style: apply formatting`)

### Type Errors

```
✗ tsc: Type error in src/api.ts
```

**Fix it:**
1. Read the type error details
2. Fix the TypeScript issue
3. Stage and commit

### Test Failures

```
✗ jest: 2 tests failed
```

**Fix it:**
1. Run tests locally to see failures
2. Fix the failing tests or the code causing failures
3. Stage and commit

## Changelog Integration

These commits enable automated changelog generation:

```markdown
## [1.2.0] - 2024-01-15

### Added
- Add rate limiting to REST endpoints (feat(api))
- Add JWT token refresh endpoint (feat(auth))

### Fixed
- Handle malformed UTF-8 sequences in parser (fix(parser))

### Changed
- Extract connection pool management (refactor(db))
```

## Quick Reference

```
feat:     New feature (MINOR)
fix:      Bug fix (PATCH)
docs:     Documentation (PATCH)
style:    Formatting (PATCH)
refactor: Restructure (PATCH)
perf:     Performance (PATCH)
test:     Tests (PATCH)
build:    Build system (PATCH)
ci:       CI config (PATCH)
chore:    Maintenance (PATCH)

! or BREAKING CHANGE: Breaking change (MAJOR) - CONFIRM WITH USER FIRST
```
