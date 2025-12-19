<overview>
All conventional commit types with SemVer impact and usage examples.
</overview>

<types>
## Commit Types

| Type | SemVer | Description | Example |
|------|--------|-------------|---------|
| `feat` | MINOR | New feature or capability | `feat(auth): add OAuth2 support` |
| `fix` | PATCH | Bug fix | `fix(parser): handle null values` |
| `docs` | PATCH | Documentation only | `docs(readme): update install steps` |
| `style` | PATCH | Formatting, whitespace | `style: fix indentation` |
| `refactor` | PATCH | Code restructure (no behavior change) | `refactor(db): extract pool logic` |
| `perf` | PATCH | Performance improvement | `perf(api): cache user lookups` |
| `test` | PATCH | Adding or fixing tests | `test(auth): add login tests` |
| `build` | PATCH | Build system changes | `build: update webpack config` |
| `ci` | PATCH | CI configuration | `ci: add GitHub Actions workflow` |
| `chore` | PATCH | Maintenance tasks | `chore: update dependencies` |
</types>

<type_selection>
## Selecting the Right Type

**Adding functionality?**
- New feature → `feat`
- Performance boost → `perf`

**Fixing something?**
- Bug fix → `fix`
- Code cleanup without behavior change → `refactor`
- Formatting only → `style`

**Documentation/config?**
- Docs update → `docs`
- Build config → `build`
- CI config → `ci`
- Tests → `test`
- Other maintenance → `chore`
</type_selection>

<breaking_changes>
## Breaking Changes

**CRITICAL: Never assume breaking. Always confirm with user first.**

Ask: "This change affects [X]. Is this a breaking change requiring MAJOR version bump?"

**Syntax options:**

```bash
# Option 1: Using ! after type
feat(api)!: change authentication endpoint

# Option 2: Footer notation
feat(api): change authentication endpoint

BREAKING CHANGE: The /auth endpoint now requires OAuth2 tokens
```

**Both trigger MAJOR version bump.**
</breaking_changes>

<scopes>
## Scope Guidelines

Use scope to clarify what's changed:

```
feat(auth): add OAuth2 support
fix(parser): handle empty input
docs(readme): update installation
refactor(db): extract connection pooling
```

**Common scopes:** `api`, `auth`, `db`, `ui`, `cli`, `config`, `deps`, `core`

**Rules:**
- Use lowercase
- Keep short (one word preferred)
- Match existing scopes when possible
- Optional but recommended for clarity
</scopes>
