<overview>
Quality guidelines, error handling, and execution rules for compound-docs skill.
</overview>

<quality_checklist>
## Quality Checklist

**Good documentation has:**

- ✅ Exact error messages (copy-paste from output)
- ✅ Specific file:line references
- ✅ Observable symptoms (what you saw, not interpretations)
- ✅ Failed attempts documented (helps avoid wrong paths)
- ✅ Technical explanation (not just "what" but "why")
- ✅ Code examples (before/after if applicable)
- ✅ Prevention guidance (how to catch early)
- ✅ Cross-references (related issues)

**Avoid:**

- ❌ Vague descriptions ("something was wrong")
- ❌ Missing technical details ("fixed the code")
- ❌ No context (which version? which file?)
- ❌ Just code dumps (explain why it works)
- ❌ No prevention guidance
- ❌ No cross-references
</quality_checklist>

<success_criteria>
## Success Criteria

Documentation is successful when ALL are true:

- ✅ YAML frontmatter validated (all required fields, correct formats)
- ✅ File created in docs/solutions/[category]/[filename].md
- ✅ Enum values match schema exactly
- ✅ Code examples included in solution section
- ✅ Cross-references added if related issues found
- ✅ User presented with decision menu and action confirmed
</success_criteria>

<execution_rules>
## Execution Rules

**MUST do:**
- Validate YAML frontmatter (BLOCK if invalid)
- Extract exact error messages from conversation
- Include code examples in solution section
- Create directories before writing files (`mkdir -p`)
- Ask user and WAIT if critical context missing

**MUST NOT do:**
- Skip YAML validation (validation gate is blocking)
- Use vague descriptions (not searchable)
- Omit code examples or cross-references
- Auto-promote to Required Reading (user decides)
</execution_rules>

<error_handling>
## Error Handling

**Missing context:**
- Ask user for missing details
- Don't proceed until critical info provided

**YAML validation failure:**
- Show specific errors
- Present retry with corrected values
- BLOCK until valid

**Similar issue ambiguity:**
- Present multiple matches
- Let user choose: new doc, update existing, or link

**Module not recognized:**
- Warn but don't block
- Proceed with documentation
- Suggest adding to module list
</error_handling>

<example>
## Example Scenario

**User:** "That worked! The N+1 query is fixed."

**Skill activates:**

1. **Detect confirmation:** "That worked!" triggers auto-invoke
2. **Gather context:**
   - Module: Brief System
   - Symptom: Brief generation taking >5 seconds
   - Failed attempts: Added pagination (didn't help)
   - Solution: Added `includes(:emails)` eager loading
   - Root cause: Missing eager loading
3. **Check existing:** No similar issue found
4. **Generate filename:** `n-plus-one-brief-generation-BriefSystem-20251110.md`
5. **Validate YAML:**
   ```yaml
   module: Brief System
   date: 2025-11-10
   problem_type: performance_issue
   severity: high
   ```
   ✅ Valid
6. **Create documentation:** `docs/solutions/performance-issues/...`
7. **Cross-reference:** None needed

**Output:** Decision menu presented
</example>
