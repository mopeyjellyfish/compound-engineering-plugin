<overview>
Quick reference commands for file-todo operations.
</overview>

<section name="finding-work">
## Finding Work

```bash
# List highest priority unblocked work
grep -l 'dependencies: \[\]' todos/*-ready-p1-*.md

# List all pending items needing triage
ls todos/*-pending-*.md

# Find next issue ID
ls todos/ | grep -o '^[0-9]\+' | sort -n | tail -1 | awk '{printf "%03d", $1+1}'

# Count by status
for status in pending ready complete; do
  echo "$status: $(ls -1 todos/*-$status-*.md 2>/dev/null | wc -l)"
done
```
</section>

<section name="dependencies">
## Dependency Management

```bash
# What blocks this todo?
grep "^dependencies:" todos/003-*.md

# What does this todo block?
grep -l 'dependencies:.*"002"' todos/*.md

# List all ready todos with no blockers
grep -l 'dependencies: \[\]' todos/*-ready-*.md
```
</section>

<section name="searching">
## Searching

```bash
# Search by tag
grep -l "tags:.*rails" todos/*.md

# Search by priority
ls todos/*-p1-*.md    # P1 (critical)
ls todos/*-p2-*.md    # P2 (important)
ls todos/*-p3-*.md    # P3 (nice-to-have)

# Full-text search
grep -r "payment" todos/

# Search by status
ls todos/*-ready-*.md
ls todos/*-pending-*.md
```
</section>

<section name="status-changes">
## Status Changes

```bash
# Pending → Ready (approved)
mv todos/002-pending-p1-fix-bug.md todos/002-ready-p1-fix-bug.md

# Ready → Complete (done)
mv todos/002-ready-p1-fix-bug.md todos/002-complete-p1-fix-bug.md

# Priority change
mv todos/002-ready-p2-refactor.md todos/002-ready-p1-refactor.md
```
</section>

<section name="creating">
## Creating Todos

```bash
# Get next ID
NEXT_ID=$(ls todos/ | grep -o '^[0-9]\+' | sort -n | tail -1 | awk '{printf "%03d", $1+1}')

# Create from template
cp assets/todo-template.md todos/${NEXT_ID}-pending-p2-description.md
```
</section>
