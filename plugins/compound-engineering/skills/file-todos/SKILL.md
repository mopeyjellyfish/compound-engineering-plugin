---
name: file-todos
description: Manages file-based todo tracking in the todos/ directory. Use for code review feedback, technical debt, feature requests, and work items with YAML frontmatter.
---

# File-Based Todo Tracking

The `todos/` directory contains markdown files for tracking code review feedback, technical debt, and work items. Each todo has YAML frontmatter and structured sections.

## When to Use

- Creating todos from findings or feedback
- Managing lifecycle (pending → ready → complete)
- Triaging pending items
- Checking/managing dependencies
- Converting PR comments to tracked work

## File Naming Convention

```
{issue_id}-{status}-{priority}-{description}.md
```

| Component | Values | Example |
|-----------|--------|---------|
| issue_id | 001, 002... (never reused) | 001 |
| status | pending, ready, complete | ready |
| priority | p1 (critical), p2, p3 | p1 |
| description | kebab-case | fix-n-plus-1 |

**Example:** `002-ready-p1-fix-n-plus-1.md`

## File Structure

**YAML frontmatter:**
```yaml
---
status: ready
priority: p1
issue_id: "002"
tags: [rails, performance]
dependencies: ["001"]  # Blocked by issue 001
---
```

**Required sections:**
- Problem Statement
- Findings
- Proposed Solutions
- Recommended Action (filled during triage)
- Acceptance Criteria
- Work Log

## Quick Workflows

| Action | Steps |
|--------|-------|
| Create todo | Get next ID → Copy template → Fill sections |
| Triage | Review pending → Approve/defer → Update status |
| Complete | Verify criteria → Update log → Rename to complete |

**Full workflows:** `references/workflows.md`

## Quick Commands

```bash
# List ready P1 work with no blockers
grep -l 'dependencies: \[\]' todos/*-ready-p1-*.md

# List pending (needs triage)
ls todos/*-pending-*.md

# Next issue ID
ls todos/ | grep -o '^[0-9]\+' | sort -n | tail -1 | awk '{printf "%03d", $1+1}'
```

**Full commands:** `references/commands.md`

## Key Distinctions

| System | Purpose |
|--------|---------|
| **File-todos (this skill)** | Markdown files in `todos/` for project tracking |
| **TodoWrite tool** | In-memory task tracking during agent sessions |
| **Database Todo model** | User-facing app feature (if exists) |

## Reference Files

| File | Content |
|------|---------|
| `references/workflows.md` | Create, triage, dependencies, complete |
| `references/commands.md` | Bash command reference |
