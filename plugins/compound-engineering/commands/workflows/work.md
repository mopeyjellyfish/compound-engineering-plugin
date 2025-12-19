---
name: workflows:work
description: Execute work plans efficiently while maintaining quality and finishing features
argument-hint: "[plan file, specification, or todo file path]"
---

# Work Plan Execution Command

Execute a work plan efficiently while maintaining quality and finishing features.

## Introduction

This command takes a work document (plan, specification, or todo file) and executes it systematically. The focus is on **shipping complete features** by understanding requirements quickly, following existing patterns, and maintaining quality throughout.

## Input Document

<input_document> #$ARGUMENTS </input_document>

## Execution Workflow

### Phase 1: Quick Start

1. **Read Plan and Clarify**

   - Read the work document completely
   - Review any references or links provided in the plan
   - If anything is unclear or ambiguous, ask clarifying questions now
   - Get user approval to proceed
   - **Do not skip this** - better to ask questions now than build the wrong thing

2. **Setup Environment**

   Choose your work style:

   **Option A: Live work on current branch**
   ```bash
   git checkout main && git pull origin main
   git checkout -b feature-branch-name
   ```

   **Option B: Parallel work with worktree (recommended for parallel development)**
   ```bash
   # Ask user first: "Work in parallel with worktree or on current branch?"
   # If worktree:
   skill: git-worktree
   # The skill will create a new branch from main in an isolated worktree
   ```

   **Recommendation**: Use worktree if:
   - You want to work on multiple features simultaneously
   - You want to keep main clean while experimenting
   - You plan to switch between branches frequently

   Use live branch if:
   - You're working on a single feature
   - You prefer staying in the main repository

3. **Create Todo List**
   - Use TodoWrite to break plan into actionable tasks
   - Include dependencies between tasks
   - Prioritize based on what needs to be done first
   - Include testing and quality check tasks
   - Keep tasks specific and completable

### Phase 2: Execute

1. **Task Execution Loop**

   For each task in priority order:

   ```
   while (tasks remain):
     - Mark task as in_progress in TodoWrite
     - Read any referenced files from the plan
     - Look for similar patterns in codebase
     - Implement following existing conventions
     - Write tests for new functionality
     - Run tests after changes
     - Mark task as completed
   ```

2. **Follow Existing Patterns**

   - The plan should reference similar code - read those files first
   - Match naming conventions exactly
   - Reuse existing components where possible
   - Follow project coding standards (see CLAUDE.md)
   - When in doubt, grep for similar implementations

3. **Test Continuously**

   - Run relevant tests after each significant change
   - Don't wait until the end to test
   - Fix failures immediately
   - Add new tests for new functionality

4. **Figma Design Sync** (if applicable)

   For UI work with Figma designs:

   - Implement components following design specs
   - Use figma-design-sync agent iteratively to compare
   - Fix visual differences identified
   - Repeat until implementation matches design

5. **Track Progress**
   - Keep TodoWrite updated as you complete tasks
   - Note any blockers or unexpected discoveries
   - Create new tasks if scope expands
   - Keep user informed of major milestones

### Phase 3: Quality Check

1. **Run Core Quality Checks**

   Always run before submitting:

   ```bash
   # Run full test suite
   bin/rails test

   # Run linting (per CLAUDE.md)
   # Use linting-agent before pushing to origin
   ```

2. **Consider Reviewer Agents** (Optional)

   Use for complex, risky, or large changes. **Select reviewers based on file types:**

   **Language-Specific Reviewers (based on files changed):**

   | File Pattern | Reviewer | Focus Area |
   |-------------|----------|------------|
   | `*.go` | **david-go-reviewer** | Go idioms, error handling, concurrency, sqlc, NATS |
   | `*.ts` | **david-typescript-reviewer** | Type safety, strict mode, generics, discriminated unions |
   | `*.tsx`, `*.jsx` | **david-react-reviewer** | Hooks, effects, component design, performance |
   | `*.tsx` (React+TS) | **Both** typescript + react reviewers | Full stack review |
   | `*.sql`, `*/sql/*` | **data-migration-expert** + **sqlc skill** | Query patterns, migration safety |
   | `*/sql/migrations/*`, `*/migrations/*` | **data-migration-expert** + **deployment-verification-agent** + **sqlc skill** | Migration rollback, deploy checklists |
   | `*/sql/queries/*`, `*/queries/*` | **david-go-reviewer** + **performance-oracle** + **sqlc skill** | sqlc queries, N+1 detection |

   **Cross-Cutting Reviewers (apply to any language):**

   | Reviewer | When to Use | Focus Area |
   |----------|------------|------------|
   | **security-sentinel** | Auth changes, APIs, user input | Injection, XSS, data exposure |
   | **performance-oracle** | Data processing, queries | N+1, complexity, caching |
   | **code-simplicity-reviewer** | After implementation (final pass) | Unnecessary complexity |
   | **architecture-strategist** | Structural changes | SOLID, boundaries, coupling |

   **How to select reviewers:**
   1. Check which file types are in your changes
   2. Run matching language-specific reviewers
   3. Add cross-cutting reviewers for risky areas
   4. Run code-simplicity-reviewer LAST as final pass

   Run reviewers in parallel with Task tool:

   ```
   # Example: TypeScript + Go changes with security concerns
   Task(david-typescript-reviewer): "Review TypeScript changes"
   Task(david-go-reviewer): "Review Go changes"
   Task(security-sentinel): "Scan for security issues"
   Task(code-simplicity-reviewer): "Final simplicity check"  # Run last
   ```

   Present findings to user and address critical issues.

3. **Final Validation**
   - All TodoWrite tasks marked completed
   - All tests pass
   - Linting passes
   - Code follows existing patterns
   - Figma designs match (if applicable)
   - No console errors or warnings

### Phase 4: Ship It

1. **Create Commit (Follow conventional-commits skill)**

   Stage and commit following the `conventional-commits` skill guidelines:

   ```bash
   git add <relevant-files>
   git status  # Review what's being committed
   git diff --staged  # Verify the changes
   ```

   Write a conventional commit message (no AI/Claude/generated references):

   ```bash
   git commit -m "$(cat <<'EOF'
   <type>(scope): description of what changed

   Optional body explaining why, not how.
   EOF
   )"
   ```

   **Pre-commit handling (CRITICAL):**
   - Read the pre-commit output completely
   - If files were auto-modified (formatting), stage and commit again
   - If pre-commit fails, fix the underlying issue - never skip or work around
   - Never use `--no-verify` or other bypass flags

   **Breaking changes:**
   - Never assume a change is breaking
   - Ask the user to confirm before using `!` or `BREAKING CHANGE:` footer
   - Only user can determine if MAJOR version bump is required

2. **Create Pull Request**

   ```bash
   git push -u origin feature-branch-name

   gh pr create --title "<type>(scope): description" --body "$(cat <<'EOF'
   ## Summary
   - What was built
   - Why it was needed
   - Key decisions made

   ## Testing
   - Tests added/modified
   - Manual testing performed
   EOF
   )"
   ```

3. **Notify User**
   - Summarize what was completed
   - Link to PR
   - Note any follow-up work needed
   - Suggest next steps if applicable

---

## Key Principles

### Start Fast, Execute Faster

- Get clarification once at the start, then execute
- Don't wait for perfect understanding - ask questions and move
- The goal is to **finish the feature**, not create perfect process

### The Plan is Your Guide

- Work documents should reference similar code and patterns
- Load those references and follow them
- Don't reinvent - match what exists

### Test As You Go

- Run tests after each change, not at the end
- Fix failures immediately
- Continuous testing prevents big surprises

### Quality is Built In

- Follow existing patterns
- Write tests for new code
- Run linting before pushing
- Use reviewer agents for complex/risky changes only

### Ship Complete Features

- Mark all tasks completed before moving on
- Don't leave features 80% done
- A finished feature that ships beats a perfect feature that doesn't

## Quality Checklist

Before creating PR, verify:

- [ ] All clarifying questions asked and answered
- [ ] All TodoWrite tasks marked completed
- [ ] Tests pass
- [ ] Linting passes
- [ ] Code follows existing patterns
- [ ] Figma designs match implementation (if applicable)
- [ ] Commit messages follow `conventional-commits` skill (no AI/Claude references)
- [ ] Pre-commit hooks pass without workarounds
- [ ] Breaking changes confirmed with user before marking as BREAKING
- [ ] PR description includes summary and testing notes

## When to Use Reviewer Agents

**Don't use by default.** Use reviewer agents only when:

- Large refactor affecting many files (10+)
- Security-sensitive changes (authentication, permissions, data access)
- Performance-critical code paths
- Complex algorithms or business logic
- User explicitly requests thorough review

For most features: tests + linting + following patterns is sufficient.

## Common Pitfalls to Avoid

- **Analysis paralysis** - Don't overthink, read the plan and execute
- **Skipping clarifying questions** - Ask now, not after building wrong thing
- **Ignoring plan references** - The plan has links for a reason
- **Testing at the end** - Test continuously or suffer later
- **Forgetting TodoWrite** - Track progress or lose track of what's done
- **80% done syndrome** - Finish the feature, don't move on early
- **Over-reviewing simple changes** - Save reviewer agents for complex work
