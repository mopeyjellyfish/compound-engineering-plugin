<overview>
Skill structure and progressive disclosure design principle.
</overview>

<structure>
## Skill Anatomy

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/      - Executable code
    ├── references/   - Documentation for context
    └── assets/       - Files for output
```
</structure>

<component name="skill-md">
## SKILL.md (required)

**Metadata quality is critical:** The `name` and `description` determine when Claude uses the skill.

- Be specific about what skill does
- Specify when to use it
- Use third person: "This skill should be used when..."
</component>

<component name="scripts">
## Scripts (`scripts/`)

Executable code for deterministic tasks.

**When to include:**
- Same code rewritten repeatedly
- Deterministic reliability needed

**Benefits:**
- Token efficient
- Deterministic
- May execute without loading into context

**Note:** Scripts may still need reading for patching or adjustments.
</component>

<component name="references">
## References (`references/`)

Documentation loaded into context as needed.

**When to include:**
- Documentation Claude should reference while working
- Database schemas, API docs, domain knowledge, policies

**Best practices:**
- Keeps SKILL.md lean
- Loaded only when Claude determines needed
- For large files (>10k words), include grep patterns in SKILL.md
- Avoid duplication: info in either SKILL.md or references, not both
</component>

<component name="assets">
## Assets (`assets/`)

Files used in output, not loaded into context.

**When to include:**
- Files used in final output
- Templates, images, icons, fonts, boilerplate

**Examples:**
- `assets/logo.png` - brand assets
- `assets/slides.pptx` - templates
- `assets/frontend-template/` - boilerplate
</component>

<progressive_disclosure>
## Progressive Disclosure

Three-level loading system for context efficiency:

| Level | Contents | Size |
|-------|----------|------|
| 1. Metadata | name + description | ~100 words |
| 2. SKILL.md body | When skill triggers | <5k words |
| 3. Bundled resources | As needed | Unlimited* |

*Scripts can execute without loading into context.
</progressive_disclosure>
