<overview>
Post-documentation decision menu. Presented after successful capture, requires user response before proceeding.
</overview>

<menu>
## Decision Menu

After successful documentation, present and WAIT for response:

```
✓ Solution documented

File created:
- docs/solutions/[category]/[filename].md

What's next?
1. Continue workflow (recommended)
2. Add to Required Reading - Promote to critical patterns
3. Link related issues - Connect to similar problems
4. Add to existing skill - Add to a learning skill
5. Create new skill - Extract into new learning skill
6. View documentation - See what was captured
7. Other
```
</menu>

<option name="continue" number="1">
## Option 1: Continue Workflow

- Return to calling skill/workflow
- Documentation is complete
- No additional action needed
</option>

<option name="required-reading" number="2">
## Option 2: Add to Required Reading

**User selects when:**
- System made this mistake multiple times across modules
- Solution is non-obvious but must be followed every time
- Foundational requirement

**Action:**
1. Extract pattern from documentation
2. Format as ❌ WRONG vs ✅ CORRECT with code examples
3. Add to `docs/solutions/patterns/critical-patterns.md`
4. Add cross-reference back to this doc
5. Confirm: "✓ Added to Required Reading"
</option>

<option name="link-issues" number="3">
## Option 3: Link Related Issues

- Prompt: "Which doc to link? (filename or describe)"
- Search docs/solutions/ for the doc
- Add cross-reference to both docs
- Confirm: "✓ Cross-reference added"
</option>

<option name="add-to-skill" number="4">
## Option 4: Add to Existing Skill

**User selects when:** Solution relates to an existing learning skill.

**Action:**
1. Prompt: "Which skill?"
2. Determine which reference file to update
3. Add link and brief description
4. Confirm: "✓ Added to [skill-name] skill"
</option>

<option name="create-skill" number="5">
## Option 5: Create New Skill

**User selects when:** Solution represents start of new learning domain.

**Action:**
1. Prompt: "What should the new skill be called?"
2. Run skill creator script
3. Create initial reference files with this solution
4. Confirm: "✓ Created new [skill-name] skill"
</option>

<option name="view-doc" number="6">
## Option 6: View Documentation

- Display the created documentation
- Present decision menu again after viewing
</option>

<option name="other" number="7">
## Option 7: Other

- Ask what they'd like to do
- Handle custom requests
</option>
