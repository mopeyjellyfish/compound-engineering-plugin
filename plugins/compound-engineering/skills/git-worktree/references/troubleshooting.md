<overview>
Common issues and solutions for git-worktree skill.
</overview>

<issue name="already-exists">
## "Worktree already exists"

**Cause:** A worktree with that name already exists.

**Solution:** Script will ask if you want to switch to it instead.

```bash
# Or manually switch:
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh switch feature-name
```
</issue>

<issue name="cannot-remove">
## "Cannot remove worktree: it is the current worktree"

**Cause:** You're trying to cleanup while inside the worktree.

**Solution:** Switch out of the worktree first:

```bash
cd $(git rev-parse --show-toplevel)
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh cleanup
```
</issue>

<issue name="lost">
## Lost in a worktree?

**Solution:** See where you are:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh list
```

Navigate back to main:

```bash
cd $(git rev-parse --show-toplevel)
```
</issue>

<issue name="missing-env">
## .env files missing in worktree

**Cause:** Worktree was created via raw `git worktree add` instead of the manager script.

**Solution:** Copy .env files:

```bash
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh copy-env feature-name
```
</issue>

<issue name="branch-conflict">
## Branch already exists on remote

**Cause:** Trying to create a worktree with a branch name that exists remotely.

**Solution:** Either use a different name or checkout the existing branch:

```bash
# Use different name
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-login-v2

# Or fetch and use existing
git fetch origin
bash ${CLAUDE_PLUGIN_ROOT}/skills/git-worktree/scripts/worktree-manager.sh create feature-login origin/feature-login
```
</issue>

<directory_structure>
## Directory Structure

```
project/
├── .worktrees/
│   ├── feature-login/          # Worktree 1
│   │   ├── .git
│   │   ├── .env                # Copied from main
│   │   └── ...
│   └── feature-notifications/  # Worktree 2
│       ├── .git
│       └── ...
└── .gitignore                  # Includes .worktrees
```
</directory_structure>

<how_it_works>
## How Worktrees Work

- Uses `git worktree add` for isolated environments
- Each worktree has its own branch
- Changes in one worktree don't affect others
- Share git history with main repo
- Can push from any worktree

**Performance:**
- Worktrees are lightweight (file system links)
- No repository duplication
- Shared git objects
- Much faster than cloning
</how_it_works>
