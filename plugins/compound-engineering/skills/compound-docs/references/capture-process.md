<overview>
The 7-step process for capturing solved problems as documentation. Each step has dependencies and some are blocking gates.
</overview>

<step name="detect-confirmation" number="1" required="true">
## Step 1: Detect Confirmation

**Auto-invoke after phrases:**
- "that worked"
- "it's fixed"
- "working now"
- "problem solved"
- "that did it"

**OR manual:** `/doc-fix` command

**Non-trivial problems only:**
- Multiple investigation attempts needed
- Tricky debugging that took time
- Non-obvious solution
- Future sessions would benefit

**Skip documentation for:**
- Simple typos
- Obvious syntax errors
- Trivial fixes immediately corrected
</step>

<step name="gather-context" number="2" required="true" depends_on="1">
## Step 2: Gather Context

Extract from conversation history:

**Required information:**
- **Module name**: Which module had the problem
- **Symptom**: Observable error/behavior (exact error messages)
- **Investigation attempts**: What didn't work and why
- **Root cause**: Technical explanation of actual problem
- **Solution**: What fixed it (code/config changes)
- **Prevention**: How to avoid in future

**Environment details:**
- Framework version
- Stage (if applicable)
- OS version
- File/line references

**BLOCKING:** If critical context missing, ask user and WAIT:

```
I need a few details to document this properly:

1. Which module had this issue?
2. What was the exact error message or symptom?
3. What stage were you in?

[Continue after user provides details]
```
</step>

<step name="check-existing" number="3" required="false" depends_on="2">
## Step 3: Check Existing Docs

Search docs/solutions/ for similar issues:

```bash
# Search by error message keywords
grep -r "exact error phrase" docs/solutions/

# Search by symptom category
ls docs/solutions/[category]/
```

**IF similar issue found:**
```
Found similar issue: docs/solutions/[path]

What's next?
1. Create new doc with cross-reference (recommended)
2. Update existing doc (only if same root cause)
3. Other

Choose (1-3): _
```

WAIT for user response, then execute chosen action.

**ELSE:** Proceed directly to Step 4.
</step>

<step name="generate-filename" number="4" required="true" depends_on="2">
## Step 4: Generate Filename

Format: `[sanitized-symptom]-[module]-[YYYYMMDD].md`

**Sanitization rules:**
- Lowercase
- Replace spaces with hyphens
- Remove special characters except hyphens
- Truncate to < 80 chars

**Examples:**
- `missing-include-BriefSystem-20251110.md`
- `parameter-not-saving-state-EmailProcessing-20251110.md`
- `webview-crash-on-resize-Assistant-20251110.md`
</step>

<step name="validate-yaml" number="5" required="true" depends_on="4" blocking="true">
## Step 5: Validate YAML Schema

**CRITICAL:** All docs require validated YAML frontmatter.

Load schema and classify problem against enum values defined in `yaml-schema.md`.

**BLOCK if validation fails:**
```
❌ YAML validation failed

Errors:
- problem_type: must be one of schema enums
- severity: must be one of [critical, moderate, minor]
- symptoms: must be array with 1-5 items

Please provide corrected values.
```

**GATE:** Do NOT proceed to Step 6 until YAML passes all validation rules.
</step>

<step name="create-documentation" number="6" required="true" depends_on="5">
## Step 6: Create Documentation

**Determine category from problem_type:** Use category mapping in `yaml-schema.md`.

**Create file:**
```bash
CATEGORY="[mapped from problem_type]"
FILENAME="[generated-filename].md"
DOC_PATH="docs/solutions/${CATEGORY}/${FILENAME}"

mkdir -p "docs/solutions/${CATEGORY}"
# Write using template from assets/resolution-template.md
```

**Result:**
- Single file in category directory
- Enum validation ensures consistent categorization
</step>

<step name="cross-reference" number="7" required="false" depends_on="6">
## Step 7: Cross-Reference & Pattern Detection

**If similar issues found in Step 3:**

Update existing doc with link:
```bash
echo "- See also: [$FILENAME]($REAL_FILE)" >> [similar-doc.md]
```

**Critical Pattern Detection:**

If issue has indicators suggesting it's critical:
- Severity: `critical` in YAML
- Affects multiple modules OR foundational stage
- Non-obvious solution

Add note in decision menu:
```
💡 This might be worth adding to Required Reading (Option 2)
```

**NEVER auto-promote.** User decides via decision menu.
</step>
