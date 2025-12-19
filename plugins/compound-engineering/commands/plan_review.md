---
name: plan_review
description: Have multiple specialized agents review a plan in parallel
argument-hint: "[plan file path or plan content]"
---

# Plan Review

Review the provided plan using specialized agents in parallel. Select reviewers based on the technologies mentioned in the plan.

## Plan to Review

<plan_content> #$ARGUMENTS </plan_content>

## Reviewer Selection

**Step 1: Detect technologies mentioned in the plan:**

Scan the plan for technology indicators:
- Go/Golang mentions → Include **david-go-reviewer**
- TypeScript/TS mentions → Include **david-typescript-reviewer**
- React/JSX/hooks mentions → Include **david-react-reviewer**
- Database/SQL/migration/sqlc mentions → Include **data-migration-expert**
- Query files (`*/sql/queries/*`) → Include **david-go-reviewer** + **performance-oracle**
- Migration files (`*/sql/migrations/*`) → Include **deployment-verification-agent**
- Security/auth mentions → Include **security-sentinel**
- Performance concerns → Include **performance-oracle**

**Step 2: Run selected reviewers in parallel:**

```
# Language-specific (based on plan content)
Task david-go-reviewer(plan) - if Go code planned
Task david-typescript-reviewer(plan) - if TypeScript planned
Task david-react-reviewer(plan) - if React planned

# Cross-cutting (always valuable for plans)
Task architecture-strategist(plan) - architectural concerns
Task security-sentinel(plan) - security implications

# Final pass (always)
Task code-simplicity-reviewer(plan) - simplification opportunities
```

**Step 3: Synthesize feedback:**

After all agents complete, synthesize their feedback into:
1. **Blockers** - Issues that must be addressed before implementation
2. **Recommendations** - Suggested improvements to the plan
3. **Approvals** - Aspects of the plan that are well-designed

Present consolidated feedback to the user.
