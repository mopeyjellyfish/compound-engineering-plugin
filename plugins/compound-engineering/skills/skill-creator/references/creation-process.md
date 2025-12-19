<overview>
Full 6-step process for creating effective skills.
</overview>

<step name="understand" number="1">
## Step 1: Understanding with Concrete Examples

Skip only when usage patterns are clearly understood.

**Questions to ask:**
- "What functionality should the skill support?"
- "Can you give examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

**Conclude when:** Clear sense of functionality the skill should support.
</step>

<step name="plan" number="2">
## Step 2: Planning Reusable Contents

Analyze each concrete example:

1. Consider how to execute from scratch
2. Identify helpful scripts, references, and assets for repeated workflows

**Examples:**

| Query | Analysis | Resource |
|-------|----------|----------|
| "Rotate this PDF" | Same code each time | `scripts/rotate_pdf.py` |
| "Build me a todo app" | Same boilerplate | `assets/hello-world/` |
| "How many users logged in?" | Rediscover schemas | `references/schema.md` |

**Output:** List of reusable resources to include.
</step>

<step name="initialize" number="3">
## Step 3: Initializing the Skill

Skip if skill already exists.

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

**Script creates:**
- Skill directory at specified path
- SKILL.md template with frontmatter and TODOs
- Example `scripts/`, `references/`, `assets/` directories

After init, customize or delete generated files as needed.
</step>

<step name="edit" number="4">
## Step 4: Edit the Skill

**Remember:** Creating for another Claude instance to use. Focus on non-obvious procedural knowledge.

**Order:**
1. Start with reusable resources (`scripts/`, `references/`, `assets/`)
2. Delete unneeded example files
3. Update SKILL.md

**SKILL.md must answer:**
1. What is the purpose? (few sentences)
2. When should it be used?
3. How should Claude use it? (reference all resources)

**Writing style:** Use imperative/infinitive form (verb-first), not second person.
- ✅ "To accomplish X, do Y"
- ❌ "You should do X"
</step>

<step name="package" number="5">
## Step 5: Packaging

```bash
scripts/package_skill.py <path/to/skill-folder>

# Optional output directory
scripts/package_skill.py <path/to/skill-folder> ./dist
```

**Script validates:**
- YAML frontmatter format
- Naming conventions
- Description quality
- File organization

**If validation passes:** Creates `skill-name.zip` for distribution.
**If validation fails:** Reports errors, fix and retry.
</step>

<step name="iterate" number="6">
## Step 6: Iterate

**Workflow:**
1. Use skill on real tasks
2. Notice struggles or inefficiencies
3. Identify updates to SKILL.md or resources
4. Implement changes
5. Test again
</step>
