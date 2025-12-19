---
name: skill-creator
description: Guide for creating effective skills. Use when users want to create or update a skill that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
license: Complete terms in LICENSE.txt
---

# Skill Creator

Guide for creating effective skills that extend Claude's capabilities.

## What Skills Provide

1. **Specialized workflows** - Multi-step procedures for domains
2. **Tool integrations** - Instructions for file formats or APIs
3. **Domain expertise** - Company-specific knowledge, schemas
4. **Bundled resources** - Scripts, references, and assets

## Skill Anatomy

```
skill-name/
├── SKILL.md (required)      - Frontmatter + instructions
├── scripts/                  - Executable code
├── references/               - Documentation for context
└── assets/                   - Files for output
```

**Full anatomy details:** `references/anatomy.md`

## Quick Creation Process

| Step | Action |
|------|--------|
| 1. Understand | Gather concrete usage examples |
| 2. Plan | Identify reusable scripts/references/assets |
| 3. Initialize | Run `scripts/init_skill.py <name>` |
| 4. Edit | Fill SKILL.md, create resources |
| 5. Package | Run `scripts/package_skill.py <path>` |
| 6. Iterate | Test, improve, repeat |

**Full process details:** `references/creation-process.md`

## Key Commands

```bash
# Initialize new skill
scripts/init_skill.py <skill-name> --path <output-dir>

# Package for distribution
scripts/package_skill.py <path/to/skill-folder>
```

## SKILL.md Requirements

**Frontmatter (required):**
```yaml
---
name: skill-name
description: When to use this skill (third person)
---
```

**Body must answer:**
1. What is the purpose?
2. When should it be used?
3. How should Claude use it?

**Writing style:** Imperative/infinitive form, not second person.
- ✅ "To accomplish X, do Y"
- ❌ "You should do X"

## Progressive Disclosure

| Level | Loaded | Size |
|-------|--------|------|
| Metadata | Always | ~100 words |
| SKILL.md body | When triggered | <5k words |
| Resources | As needed | Unlimited |

## Reference Files

| File | Content |
|------|---------|
| `references/anatomy.md` | Skill structure, resource types |
| `references/creation-process.md` | Full 6-step process |
