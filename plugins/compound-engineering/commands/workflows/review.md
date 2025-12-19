---
name: workflows:review
description: Perform exhaustive code reviews using multi-agent analysis, ultra-thinking, and worktrees
argument-hint: "[PR number, GitHub URL, branch name, or latest]"
---

# Review Command

<command_purpose> Perform exhaustive code reviews using multi-agent analysis, ultra-thinking, and Git worktrees for deep local inspection. </command_purpose>

## Introduction

<role>Senior Code Review Architect with expertise in security, performance, architecture, and quality assurance</role>

## Prerequisites

<requirements>
- Git repository with GitHub CLI (`gh`) installed and authenticated
- Clean main/master branch
- Proper permissions to create worktrees and access the repository
- For document reviews: Path to a markdown file or document
</requirements>

## Main Tasks

### 1. Determine Review Target & Setup (ALWAYS FIRST)

<review_target> #$ARGUMENTS </review_target>

<thinking>
First, I need to determine the review target type and set up the code for analysis.
</thinking>

#### Immediate Actions:

<task_list>

- [ ] Determine review type: PR number (numeric), GitHub URL, file path (.md), or empty (current branch)
- [ ] Check current git branch
- [ ] If ALREADY on the PR branch → proceed with analysis on current branch
- [ ] If DIFFERENT branch → offer to use worktree for isolated review, invoke `skill: git-worktree` with branch name
- [ ] Fetch PR metadata using `gh pr view --json` for title, body, files, linked issues
- [ ] Set up language-specific analysis tools
- [ ] Prepare security scanning environment
- [ ] Make sure we are on the branch we are reviewing. Use gh pr checkout to switch to the branch or manually checkout the branch.

Ensure that the code is ready for analysis (either in worktree or on current branch). ONLY then proceed to the next step.

</task_list>

#### Step 1: Detect Languages and Technologies from PR Files

<language_detection>

**CRITICAL:** Before invoking reviewers, analyze the PR file list to determine which technologies are present. This ensures you invoke the RIGHT reviewers for the actual code changes.

Run this detection logic:

```
For each file in PR:
  - *.go                              → GO_CODE = true
  - *.ts, *.tsx                       → TYPESCRIPT_CODE = true
  - *.tsx, *.jsx                      → REACT_CODE = true
  - *.rb                              → RUBY_CODE = true
  - db/migrate/*                      → DATABASE_MIGRATION = true
  - */sql/migrations/*, */migrations/* → DATABASE_MIGRATION = true
  - */sql/queries/*, */queries/*      → SQL_QUERIES = true
  - */sql/*.sql, *.sql                → SQL_CODE = true
  - Dockerfile, *.yaml                → DEVOPS_CODE = true
  - package.json changes              → DEPENDENCY_CHANGES = true
```

</language_detection>

#### Step 2: Select and Run Language-Specific Reviewers

<reviewer_selection_matrix>

**IMPORTANT:** Use this matrix to select which reviewers to run based on detected file types. Run selected reviewers in parallel using the Task tool.

| File Patterns | Required Reviewer(s) | Description |
|--------------|---------------------|-------------|
| `*.go` | **david-go-reviewer** | Go code: idiomatic patterns, error handling, concurrency, sqlc, NATS, connect-rpc |
| `*.ts` (non-React) | **david-typescript-reviewer** | TypeScript: strict mode, type safety, discriminated unions, generics |
| `*.tsx`, `*.jsx` | **david-react-reviewer** | React: hooks patterns, effects hygiene, component design, performance |
| `*.tsx` (with TS logic) | **david-typescript-reviewer** + **david-react-reviewer** | Run BOTH for React+TypeScript |
| `*.sql`, `*/sql/*` | **data-migration-expert** + **data-integrity-guardian** | Database: query patterns, migration safety, ID mappings |
| `*/sql/migrations/*`, `*/migrations/*`, `db/migrate/*` | **data-migration-expert** + **data-integrity-guardian** + **deployment-verification-agent** | Migration files: rollback safety, deployment checklists |
| `*/sql/queries/*`, `*/queries/*` | **david-go-reviewer** (for sqlc) + **performance-oracle** + **sqlc skill** | Query files: N+1 detection, query optimization, sqlc best practices |

</reviewer_selection_matrix>

#### Step 3: Run Cross-Cutting Reviewers (Always Applicable)

<cross_cutting_reviewers>

These reviewers apply to ALL code changes regardless of language. Run them in parallel with language-specific reviewers:

| Reviewer | When to Use | Focus Area |
|----------|------------|------------|
| **security-sentinel** | ALWAYS for any code changes | Auth, input validation, injection attacks, data exposure |
| **performance-oracle** | New features, data processing, queries | Algorithmic complexity, N+1 queries, memory, caching |
| **architecture-strategist** | Refactors, new services, structural changes | Component boundaries, SOLID, coupling/cohesion |
| **pattern-recognition-specialist** | Large PRs, refactoring | Code duplication, anti-patterns, naming consistency |
| **agent-native-reviewer** | New features (ALWAYS) | Action parity, context parity for AI agents |

</cross_cutting_reviewers>

#### Step 4: Run Conditional Reviewers Based on Change Type

<conditional_reviewers>

**Database/Migration Changes** (if `*.sql`, `db/migrate/*`, or migration-related changes):

| Reviewer | Trigger | What It Checks |
|----------|---------|----------------|
| **data-migration-expert** | Migrations, ID mappings, enum changes | ID mappings match production, swapped values, orphaned data |
| **data-integrity-guardian** | Schema changes, data models | Constraints, referential integrity, GDPR compliance |
| **deployment-verification-agent** | Risky data changes, production-touching | Pre/post-deploy checklists, SQL verification, rollback |

**When to run database reviewers:**
- PR includes migration files: `db/migrate/*`, `*/sql/migrations/*`, `*/migrations/*`
- PR includes query files: `*/sql/queries/*`, `*/queries/*`, `*.sql`
- PR modifies columns storing IDs, enums, or mappings
- PR includes data backfill scripts
- PR title/body mentions: migration, backfill, data transformation, sqlc

**DevOps/Infrastructure Changes** (if `Dockerfile`, `*.yaml`, CI configs):

| Reviewer | Trigger | What It Checks |
|----------|---------|----------------|
| **devops-harmony-analyst** | Docker, CI/CD, deployment configs | Build reproducibility, security, deployment safety |

</conditional_reviewers>

#### Step 5: Parallel Agent Execution

<parallel_execution>

Based on detection above, construct your parallel Task calls. Example for a mixed TS + Go + SQL PR:

```
# Language-specific (run based on detected files)
Task david-typescript-reviewer(PR content)    # If *.ts files
Task david-react-reviewer(PR content)         # If *.tsx/*.jsx files
Task david-go-reviewer(PR content)            # If *.go files
Task data-migration-expert(PR content)        # If *.sql/migrations

# SQL/sqlc specific (if */sql/* files present)
skill: sqlc                                   # Invoke for query/migration best practices

# Cross-cutting (always run these)
Task security-sentinel(PR content)
Task performance-oracle(PR content)
Task architecture-strategist(PR content)
Task agent-native-reviewer(PR content)
Task pattern-recognition-specialist(PR content)
```

**Execution Order:**
1. Run ALL language-specific + cross-cutting reviewers in PARALLEL
2. Run **code-simplicity-reviewer** LAST (after all other reviews complete) as a final simplification pass

</parallel_execution>

#### Quick Reference: File Pattern → Reviewer Mapping

<quick_reference>

```
*.go                    → david-go-reviewer
*.ts                    → david-typescript-reviewer
*.tsx/*.jsx             → david-react-reviewer (+ typescript-reviewer if TS)
*.sql, */sql/*          → data-migration-expert, data-integrity-guardian
*/sql/migrations/*      → + deployment-verification-agent
*/migrations/*          → + deployment-verification-agent
db/migrate/*            → + deployment-verification-agent
*/sql/queries/*         → david-go-reviewer, performance-oracle, + invoke sqlc skill
*/queries/*             → david-go-reviewer, performance-oracle, + invoke sqlc skill
Dockerfile              → devops-harmony-analyst
*.yaml (CI)             → devops-harmony-analyst
package.json            → dependency-detective
ANY CODE                → security-sentinel, performance-oracle, architecture-strategist
NEW FEATURES            → agent-native-reviewer
FINAL PASS              → code-simplicity-reviewer (run LAST)
```

</quick_reference>

### 4. Ultra-Thinking Deep Dive Phases

<ultrathink_instruction> For each phase below, spend maximum cognitive effort. Think step by step. Consider all angles. Question assumptions. And bring all reviews in a synthesis to the user.</ultrathink_instruction>

<deliverable>
Complete system context map with component interactions
</deliverable>

#### Phase 3: Stakeholder Perspective Analysis

<thinking_prompt> ULTRA-THINK: Put yourself in each stakeholder's shoes. What matters to them? What are their pain points? </thinking_prompt>

<stakeholder_perspectives>

1. **Developer Perspective** <questions>

   - How easy is this to understand and modify?
   - Are the APIs intuitive?
   - Is debugging straightforward?
   - Can I test this easily? </questions>

2. **Operations Perspective** <questions>

   - How do I deploy this safely?
   - What metrics and logs are available?
   - How do I troubleshoot issues?
   - What are the resource requirements? </questions>

3. **End User Perspective** <questions>

   - Is the feature intuitive?
   - Are error messages helpful?
   - Is performance acceptable?
   - Does it solve my problem? </questions>

4. **Security Team Perspective** <questions>

   - What's the attack surface?
   - Are there compliance requirements?
   - How is data protected?
   - What are the audit capabilities? </questions>

5. **Business Perspective** <questions>
   - What's the ROI?
   - Are there legal/compliance risks?
   - How does this affect time-to-market?
   - What's the total cost of ownership? </questions> </stakeholder_perspectives>

#### Phase 4: Scenario Exploration

<thinking_prompt> ULTRA-THINK: Explore edge cases and failure scenarios. What could go wrong? How does the system behave under stress? </thinking_prompt>

<scenario_checklist>

- [ ] **Happy Path**: Normal operation with valid inputs
- [ ] **Invalid Inputs**: Null, empty, malformed data
- [ ] **Boundary Conditions**: Min/max values, empty collections
- [ ] **Concurrent Access**: Race conditions, deadlocks
- [ ] **Scale Testing**: 10x, 100x, 1000x normal load
- [ ] **Network Issues**: Timeouts, partial failures
- [ ] **Resource Exhaustion**: Memory, disk, connections
- [ ] **Security Attacks**: Injection, overflow, DoS
- [ ] **Data Corruption**: Partial writes, inconsistency
- [ ] **Cascading Failures**: Downstream service issues </scenario_checklist>

### 6. Multi-Angle Review Perspectives

#### Technical Excellence Angle

- Code craftsmanship evaluation
- Engineering best practices
- Technical documentation quality
- Tooling and automation assessment

#### Business Value Angle

- Feature completeness validation
- Performance impact on users
- Cost-benefit analysis
- Time-to-market considerations

#### Risk Management Angle

- Security risk assessment
- Operational risk evaluation
- Compliance risk verification
- Technical debt accumulation

#### Team Dynamics Angle

- Code review etiquette
- Knowledge sharing effectiveness
- Collaboration patterns
- Mentoring opportunities

### 4. Simplification and Minimalism Review

Run the Task code-simplicity-reviewer() to see if we can simplify the code.

### 5. Findings Synthesis and Todo Creation Using file-todos Skill

<critical_requirement> ALL findings MUST be stored in the todos/ directory using the file-todos skill. Create todo files immediately after synthesis - do NOT present findings for user approval first. Use the skill for structured todo management. </critical_requirement>

#### Step 1: Synthesize All Findings

<thinking>
Consolidate all agent reports into a categorized list of findings.
Remove duplicates, prioritize by severity and impact.
</thinking>

<synthesis_tasks>

- [ ] Collect findings from all parallel agents
- [ ] Categorize by type: security, performance, architecture, quality, etc.
- [ ] Assign severity levels: 🔴 CRITICAL (P1), 🟡 IMPORTANT (P2), 🔵 NICE-TO-HAVE (P3)
- [ ] Remove duplicate or overlapping findings
- [ ] Estimate effort for each finding (Small/Medium/Large)

</synthesis_tasks>

#### Step 2: Create Todo Files Using file-todos Skill

<critical_instruction> Use the file-todos skill to create todo files for ALL findings immediately. Do NOT present findings one-by-one asking for user approval. Create all todo files in parallel using the skill, then summarize results to user. </critical_instruction>

**Implementation Options:**

**Option A: Direct File Creation (Fast)**

- Create todo files directly using Write tool
- All findings in parallel for speed
- Use standard template from `.claude/skills/file-todos/assets/todo-template.md`
- Follow naming convention: `{issue_id}-pending-{priority}-{description}.md`

**Option B: Sub-Agents in Parallel (Recommended for Scale)** For large PRs with 15+ findings, use sub-agents to create finding files in parallel:

```bash
# Launch multiple finding-creator agents in parallel
Task() - Create todos for first finding
Task() - Create todos for second finding
Task() - Create todos for third finding
etc. for each finding.
```

Sub-agents can:

- Process multiple findings simultaneously
- Write detailed todo files with all sections filled
- Organize findings by severity
- Create comprehensive Proposed Solutions
- Add acceptance criteria and work logs
- Complete much faster than sequential processing

**Execution Strategy:**

1. Synthesize all findings into categories (P1/P2/P3)
2. Group findings by severity
3. Launch 3 parallel sub-agents (one per severity level)
4. Each sub-agent creates its batch of todos using the file-todos skill
5. Consolidate results and present summary

**Process (Using file-todos Skill):**

1. For each finding:

   - Determine severity (P1/P2/P3)
   - Write detailed Problem Statement and Findings
   - Create 2-3 Proposed Solutions with pros/cons/effort/risk
   - Estimate effort (Small/Medium/Large)
   - Add acceptance criteria and work log

2. Use file-todos skill for structured todo management:

   ```bash
   skill: file-todos
   ```

   The skill provides:

   - Template location: `.claude/skills/file-todos/assets/todo-template.md`
   - Naming convention: `{issue_id}-{status}-{priority}-{description}.md`
   - YAML frontmatter structure: status, priority, issue_id, tags, dependencies
   - All required sections: Problem Statement, Findings, Solutions, etc.

3. Create todo files in parallel:

   ```bash
   {next_id}-pending-{priority}-{description}.md
   ```

4. Examples:

   ```
   001-pending-p1-path-traversal-vulnerability.md
   002-pending-p1-api-response-validation.md
   003-pending-p2-concurrency-limit.md
   004-pending-p3-unused-parameter.md
   ```

5. Follow template structure from file-todos skill

**Todo File Structure (from template):**

Each todo must include:

- **YAML frontmatter**: status, priority, issue_id, tags, dependencies
- **Problem Statement**: What's broken/missing, why it matters
- **Findings**: Discoveries from agents with evidence/location
- **Proposed Solutions**: 2-3 options, each with pros/cons/effort/risk
- **Recommended Action**: (Filled during triage, leave blank initially)
- **Technical Details**: Affected files, components, database changes
- **Acceptance Criteria**: Testable checklist items
- **Work Log**: Dated record with actions and learnings
- **Resources**: Links to PR, issues, documentation, similar patterns

**File naming convention:**

```
{issue_id}-{status}-{priority}-{description}.md

Examples:
- 001-pending-p1-security-vulnerability.md
- 002-pending-p2-performance-optimization.md
- 003-pending-p3-code-cleanup.md
```

**Status values:**

- `pending` - New findings, needs triage/decision
- `ready` - Approved by manager, ready to work
- `complete` - Work finished

**Priority values:**

- `p1` - Critical (blocks merge, security/data issues)
- `p2` - Important (should fix, architectural/performance)
- `p3` - Nice-to-have (enhancements, cleanup)

**Tagging:** Always add `code-review` tag, plus: `security`, `performance`, `architecture`, `rails`, `quality`, etc.

#### Step 3: Summary Report

After creating all todo files, present comprehensive summary:

````markdown
## ✅ Code Review Complete

**Review Target:** PR #XXXX - [PR Title] **Branch:** [branch-name]

### Findings Summary:

- **Total Findings:** [X]
- **🔴 CRITICAL (P1):** [count] - BLOCKS MERGE
- **🟡 IMPORTANT (P2):** [count] - Should Fix
- **🔵 NICE-TO-HAVE (P3):** [count] - Enhancements

### Created Todo Files:

**P1 - Critical (BLOCKS MERGE):**

- `001-pending-p1-{finding}.md` - {description}
- `002-pending-p1-{finding}.md` - {description}

**P2 - Important:**

- `003-pending-p2-{finding}.md` - {description}
- `004-pending-p2-{finding}.md` - {description}

**P3 - Nice-to-Have:**

- `005-pending-p3-{finding}.md` - {description}

### Review Agents Used:

- david-go-reviewer / david-typescript-reviewer / david-react-reviewer (based on files)
- security-sentinel
- performance-oracle
- architecture-strategist
- agent-native-reviewer
- [other agents]

### Next Steps:

1. **Address P1 Findings**: CRITICAL - must be fixed before merge

   - Review each P1 todo in detail
   - Implement fixes or request exemption
   - Verify fixes before merging PR

2. **Triage All Todos**:
   ```bash
   ls todos/*-pending-*.md  # View all pending todos
   /triage                  # Use slash command for interactive triage
   ```
````

3. **Work on Approved Todos**:

   ```bash
   /resolve_todo_parallel  # Fix all approved items efficiently
   ```

4. **Track Progress**:
   - Rename file when status changes: pending → ready → complete
   - Update Work Log as you work
   - Commit todos: `git add todos/ && git commit -m "refactor: add code review findings"`

### Severity Breakdown:

**🔴 P1 (Critical - Blocks Merge):**

- Security vulnerabilities
- Data corruption risks
- Breaking changes
- Critical architectural issues

**🟡 P2 (Important - Should Fix):**

- Performance issues
- Significant architectural concerns
- Major code quality problems
- Reliability issues

**🔵 P3 (Nice-to-Have):**

- Minor improvements
- Code cleanup
- Optimization opportunities
- Documentation updates

```

### 7. End-to-End Testing (Optional)

<detect_project_type>

**First, detect the project type from PR files:**

| Indicator | Project Type |
|-----------|--------------|
| `*.xcodeproj`, `*.xcworkspace`, `Package.swift` (iOS) | iOS/macOS |
| `Gemfile`, `package.json`, `app/views/*`, `*.html.*` | Web |
| Both iOS files AND web files | Hybrid (test both) |

</detect_project_type>

<offer_testing>

After presenting the Summary Report, offer appropriate testing based on project type:

**For Web Projects:**
```markdown
**"Want to run Playwright browser tests on the affected pages?"**
1. Yes - run `/playwright-test`
2. No - skip
```

**For iOS Projects:**
```markdown
**"Want to run Xcode simulator tests on the app?"**
1. Yes - run `/xcode-test`
2. No - skip
```

**For Hybrid Projects (e.g., Rails + Hotwire Native):**
```markdown
**"Want to run end-to-end tests?"**
1. Web only - run `/playwright-test`
2. iOS only - run `/xcode-test`
3. Both - run both commands
4. No - skip
```

</offer_testing>

#### If User Accepts Web Testing:

Spawn a subagent to run Playwright tests (preserves main context):

```
Task general-purpose("Run /playwright-test for PR #[number]. Test all affected pages, check for console errors, handle failures by creating todos and fixing.")
```

The subagent will:
1. Identify pages affected by the PR
2. Navigate to each page and capture snapshots
3. Check for console errors
4. Test critical interactions
5. Pause for human verification on OAuth/email/payment flows
6. Create P1 todos for any failures
7. Fix and retry until all tests pass

**Standalone:** `/playwright-test [PR number]`

#### If User Accepts iOS Testing:

Spawn a subagent to run Xcode tests (preserves main context):

```
Task general-purpose("Run /xcode-test for scheme [name]. Build for simulator, install, launch, take screenshots, check for crashes.")
```

The subagent will:
1. Verify XcodeBuildMCP is installed
2. Discover project and schemes
3. Build for iOS Simulator
4. Install and launch app
5. Take screenshots of key screens
6. Capture console logs for errors
7. Pause for human verification (Sign in with Apple, push, IAP)
8. Create P1 todos for any failures
9. Fix and retry until all tests pass

**Standalone:** `/xcode-test [scheme]`

### Important: P1 Findings Block Merge

Any **🔴 P1 (CRITICAL)** findings must be addressed before merging the PR. Present these prominently and ensure they're resolved before accepting the PR.
```
