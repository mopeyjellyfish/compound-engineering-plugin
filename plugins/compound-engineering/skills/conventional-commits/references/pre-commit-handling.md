<overview>
How to handle pre-commit hooks properly. Never bypass, always fix the underlying issue.
</overview>

<critical_rules>
## Critical Rules

**NEVER use:**
- `--no-verify` to skip hooks
- `--no-gpg-sign` unless explicitly required
- Workarounds that "trick" pre-commit into passing

**ALWAYS:**
- Read pre-commit output completely
- Fix the actual problem
- Stage fixes and commit again
</critical_rules>

<workflow>
## Pre-Commit Workflow

After running `git commit`:

1. **Read the output completely**
2. **Check for auto-modifications** (formatting, linting fixes)
3. **If files were modified:**
   ```bash
   git add .
   git commit -m "style: apply auto-formatting"
   ```
4. **If pre-commit fails:**
   - Read the error message
   - Fix the actual problem
   - Stage the fix
   - Commit again with original message
</workflow>

<error_linting>
## Linting Errors

```
✗ eslint: Found 3 errors
```

**Fix it:**
1. Read the specific errors shown
2. Fix each error in source code
3. Stage fixes: `git add .`
4. Commit again with same message
</error_linting>

<error_formatting>
## Formatting Changes

```
✓ prettier: Fixed 2 files
```

**Handle it:**
1. Pre-commit auto-fixed files
2. Stage changes: `git add .`
3. Commit again (same message or `style: apply formatting`)
</error_formatting>

<error_types>
## Type Errors

```
✗ tsc: Type error in src/api.ts
```

**Fix it:**
1. Read the type error details
2. Fix the TypeScript issue
3. Stage and commit
</error_types>

<error_tests>
## Test Failures

```
✗ jest: 2 tests failed
```

**Fix it:**
1. Run tests locally to see failures
2. Fix failing tests or code causing failures
3. Stage and commit
</error_tests>
