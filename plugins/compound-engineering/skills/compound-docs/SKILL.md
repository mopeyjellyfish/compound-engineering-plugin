---
name: compound-docs
description: Capture solved problems as categorized documentation with YAML frontmatter for fast lookup
allowed-tools:
  - Read # Parse conversation context
  - Write # Create resolution docs
  - Bash # Create directories
  - Grep # Search existing docs
preconditions:
  - Problem has been solved (not in-progress)
  - Solution has been verified working
---

# compound-docs Skill

Automatically document solved problems to build searchable institutional knowledge.

**Organization:** Single-file per problem in category directories (e.g., `docs/solutions/performance-issues/n-plus-one-briefs.md`). Files use YAML frontmatter for metadata and searchability.

## When to Trigger

**Auto-invoke after confirmation phrases:**
- "that worked", "it's fixed", "working now", "problem solved"

**Manual:** `/doc-fix` command

**Document when:**
- Multiple investigation attempts needed
- Tricky debugging that took time
- Non-obvious solution
- Future sessions would benefit

**Skip for:** Simple typos, obvious syntax errors, trivial fixes

## 7-Step Process (Summary)

| Step | Action | Blocking? |
|------|--------|-----------|
| 1 | Detect confirmation | - |
| 2 | Gather context (module, symptom, solution) | Ask if missing |
| 3 | Check existing docs for similar issues | - |
| 4 | Generate filename | - |
| 5 | Validate YAML against schema | **BLOCKING** |
| 6 | Create documentation file | - |
| 7 | Cross-reference & pattern detection | - |

**Full process details:** `references/capture-process.md`

## Quick Reference

### Required Context (Step 2)

- **Module name** - Which module had the problem
- **Symptom** - Exact error messages, observable behavior
- **Investigation attempts** - What didn't work and why
- **Root cause** - Technical explanation
- **Solution** - What fixed it (code/config)
- **Prevention** - How to avoid in future

### Filename Format (Step 4)

```
[sanitized-symptom]-[module]-[YYYYMMDD].md
```

Example: `n-plus-one-brief-generation-BriefSystem-20251110.md`

### YAML Validation (Step 5)

**BLOCKING GATE:** Must validate against schema before creating file.

Required fields: `module`, `date`, `problem_type`, `severity`, `symptoms`

**Schema reference:** `references/yaml-schema.md`

## Post-Documentation Menu

After capture, present options and WAIT:

```
1. Continue workflow (recommended)
2. Add to Required Reading
3. Link related issues
4. Add to existing skill
5. Create new skill
6. View documentation
7. Other
```

**Full menu details:** `references/decision-menu.md`

## Integration

**Invoked by:**
- `/compound` command
- Manual after solution confirmed
- Auto-detection of confirmation phrases

**Invokes:** None (terminal skill)

## Reference Files

| File | Content |
|------|---------|
| `references/capture-process.md` | Full 7-step process with examples |
| `references/decision-menu.md` | Post-documentation options |
| `references/quality-guidelines.md` | Quality checklist, error handling |
| `references/yaml-schema.md` | YAML frontmatter schema |
