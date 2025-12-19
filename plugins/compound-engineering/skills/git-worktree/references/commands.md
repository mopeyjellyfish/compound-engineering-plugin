<overview>
Full command reference for worktree-manager.sh script.
</overview>

<command name="create">
## create

Creates a new worktree with the given branch name.

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create <branch-name> [from-branch]
```

**Arguments:**
- `branch-name` (required): Name for the new branch and worktree
- `from-branch` (optional): Base branch (defaults to `main`)

**Example:**
```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-login
```

**What happens:**
1. Checks if worktree already exists
2. Updates base branch from remote
3. Creates new worktree and branch
4. Copies all .env files (.env, .env.local, .env.test)
5. Shows path for cd-ing to worktree
</command>

<command name="list">
## list / ls

Lists all available worktrees with status.

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh list
```

**Output shows:**
- Worktree name
- Branch name
- Current worktree (marked with ✓)
- Main repo status
</command>

<command name="switch">
## switch / go

Switches to an existing worktree.

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh switch <name>
```

**If name not provided:** Lists available worktrees and prompts for selection.
</command>

<command name="copy-env">
## copy-env

Copies .env files to an existing worktree.

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh copy-env <name>
```

**Use when:** Worktree was created without .env files (e.g., via raw `git worktree add`).
</command>

<command name="cleanup">
## cleanup / clean

Interactively cleans up inactive worktrees.

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh cleanup
```

**What happens:**
1. Lists all inactive worktrees
2. Asks for confirmation
3. Removes selected worktrees
4. Cleans up empty directories

**Safety:** Won't remove current worktree.
</command>

<workflow_examples>
## Workflow Examples

### Code Review

```bash
# Create worktree for PR review
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create pr-123-feature

# Work in isolated worktree
cd .worktrees/pr-123-feature

# Return and cleanup
cd ../..
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh cleanup
```

### Parallel Development

```bash
# Create multiple feature worktrees
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-login
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-notifications

# List and switch between them
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh list
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh switch feature-login
```
</workflow_examples>
