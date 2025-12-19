---
name: git-worktree
description: Manages Git worktrees for isolated parallel development. Handles creating, listing, switching, and cleaning up worktrees with automatic .env copying.
---

# Git Worktree Manager

Unified interface for managing Git worktrees across development workflow.

## CRITICAL: Always Use the Manager Script

**NEVER call `git worktree add` directly.**

```bash
# ✅ CORRECT
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-name

# ❌ WRONG
git worktree add .worktrees/feature-name -b feature-name main
```

The script handles: .env copying, .gitignore management, consistent structure.

## When to Use

| Scenario | Action |
|----------|--------|
| `/workflows:review` | Offer worktree if not on PR branch |
| `/workflows:work` | Ask: live branch or worktree? |
| Parallel development | Multiple features simultaneously |
| Cleanup | After completing work |

## Quick Commands

```bash
# Create (copies .env files automatically)
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-login

# List all worktrees
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh list

# Switch to worktree
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh switch feature-login

# Cleanup completed worktrees
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh cleanup
```

**Full command reference:** `references/commands.md`

## Workflow Integration

### /workflows:review

```
1. Check current branch
2. If ALREADY on PR branch → stay, no worktree needed
3. If DIFFERENT branch → offer:
   "Use worktree for isolated review? (y/n)"
```

### /workflows:work

```
"How do you want to work?
 1. New branch on current worktree (live work)
 2. Worktree (parallel work)"
```

## Design Principles

- **KISS:** One script handles all operations
- **Opinionated:** Worktrees from `main`, stored in `.worktrees/`
- **Safe:** Confirms before create/cleanup, won't remove current

## Common Issues

| Issue | Solution |
|-------|----------|
| "Worktree already exists" | Script offers to switch |
| "Cannot remove current worktree" | `cd` out first, then cleanup |
| Missing .env files | `copy-env feature-name` |

**Full troubleshooting:** `references/troubleshooting.md`

## Reference Files

| File | Content |
|------|---------|
| `references/commands.md` | Full command reference with examples |
| `references/troubleshooting.md` | Common issues and solutions |
