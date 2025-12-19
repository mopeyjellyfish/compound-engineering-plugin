<overview>
Detailed workflows for creating, triaging, and completing file-based todos.
</overview>

<workflow name="create">
## Creating a New Todo

1. Determine next issue ID:
   ```bash
   ls todos/ | grep -o '^[0-9]\+' | sort -n | tail -1 | awk '{printf "%03d", $1+1}'
   ```

2. Copy template:
   ```bash
   cp assets/todo-template.md todos/{NEXT_ID}-pending-{priority}-{description}.md
   ```

3. Fill required sections:
   - Problem Statement
   - Findings (if from investigation)
   - Proposed Solutions (multiple options)
   - Acceptance Criteria
   - Initial Work Log entry

4. Determine status:
   - `pending` - needs triage/approval
   - `ready` - pre-approved, can start immediately

5. Add relevant tags for filtering

**When to create todo:**
- Requires > 15-20 minutes of work
- Needs research or planning
- Has dependencies on other work
- Requires manager approval
- Part of larger feature/refactor
- Technical debt needing documentation

**When to act immediately:**
- Trivial (< 15 minutes)
- Complete context available now
- No planning needed
- User explicitly requests immediate action
- Simple bug fix with obvious solution
</workflow>

<workflow name="triage">
## Triaging Pending Items

1. List pending items:
   ```bash
   ls todos/*-pending-*.md
   ```

2. For each todo:
   - Read Problem Statement and Findings
   - Review Proposed Solutions
   - Decide: approve, defer, or modify priority

3. Update approved todos:
   ```bash
   # Rename file
   mv {file}-pending-{pri}-{desc}.md {file}-ready-{pri}-{desc}.md
   ```
   - Update frontmatter: `status: pending` → `status: ready`
   - Fill "Recommended Action" section
   - Adjust priority if needed

4. Deferred todos stay `pending`

**Slash command:** `/triage` for interactive approval
</workflow>

<workflow name="dependencies">
## Managing Dependencies

**Track dependencies in frontmatter:**
```yaml
dependencies: ["002", "005"]  # Blocked by 002 and 005
dependencies: []               # No blockers
```

**Check what blocks a todo:**
```bash
grep "^dependencies:" todos/003-*.md
```

**Find what a todo blocks:**
```bash
grep -l 'dependencies:.*"002"' todos/*.md
```

**Verify blockers are complete:**
```bash
for dep in 001 002 003; do
  [ -f "todos/${dep}-complete-*.md" ] || echo "Issue $dep not complete"
done
```
</workflow>

<workflow name="work-log">
## Updating Work Logs

**Always add entry when working:**

```markdown
### YYYY-MM-DD - Session Title

**By:** Claude Code / Developer Name

**Actions:**
- Specific changes made (file:line references)
- Commands executed
- Tests run
- Investigation results

**Learnings:**
- What worked / didn't
- Patterns discovered
- Key insights for future
```

**Work logs serve as:**
- Historical record of investigation
- Documentation of approaches tried
- Knowledge sharing for team
- Context for future similar work
</workflow>

<workflow name="complete">
## Completing a Todo

1. Verify all acceptance criteria checked off
2. Update Work Log with final session
3. Rename file:
   ```bash
   mv {file}-ready-{pri}-{desc}.md {file}-complete-{pri}-{desc}.md
   ```
4. Update frontmatter: `status: complete`
5. Check for unblocked work:
   ```bash
   grep -l 'dependencies:.*"002"' todos/*-ready-*.md
   ```
6. Commit: `feat: resolve issue 002`
</workflow>
