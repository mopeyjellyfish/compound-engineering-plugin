# Changelog

All notable changes to the compound-engineering plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.1.0] - 2025-12-19

### Added

- **`conventional-commits` skill** - Ensures all commits follow the conventional commits specification v1.0.0 with proper pre-commit handling and semantic versioning alignment. Includes:
  - Full commit type reference (feat, fix, docs, style, refactor, perf, test, build, ci, chore)
  - Breaking change workflow requiring user confirmation before MAJOR version bumps
  - Pre-commit hook handling without workarounds or bypasses
  - HEREDOC templates for multi-line commit messages
  - Changelog integration patterns

### Changed

- **`/workflows:work` command** - Updated Phase 4 (Ship It) to reference `conventional-commits` skill. Removed AI/Claude/generated references from commit messages and PR descriptions. Added pre-commit handling requirements and breaking change confirmation workflow.

## [3.0.0] - 2025-12-19

### Breaking Changes

This release refocuses the plugin on **Go, React, and TypeScript stacks**, removing Ruby/Rails/Python-specific components. If you rely on the removed components, stay on version 2.16.0.

### Added

- **`david-go-reviewer` agent** - Go code review with idiomatic patterns and DHH's philosophy of simplicity. Specialized for stacks using NATS, sqlc, connect-rpc, PostgreSQL, WASM, and Goja. Covers error handling, concurrency patterns, interface design, DDD architecture, and thorough coverage of the Go ecosystem.
- **`david-react-reviewer` agent** - React code review with strict conventions. Focuses on hooks patterns (dependency arrays, custom hook design, rules of hooks), component design (single responsibility, props design, discriminated unions), state management (useState vs useReducer, derived state), performance optimization (proper memoization, re-render analysis), and effects hygiene (cleanup functions, AbortController).
- **`david-typescript-reviewer` agent** - TypeScript code review with strict type safety focus. Enforces strict mode configuration, eliminates `any` usage in favor of `unknown` with type guards, validates type inference patterns, reviews discriminated unions for domain modeling, checks generic constraints, ensures null safety, and promotes advanced patterns like template literal types and const assertions.

### Changed

- **`ankane-readme-writer` agent** → **`concise-readme-writer` agent** - Converted from Ruby gem-specific to language-agnostic. Now works for Go, TypeScript, Python, Rust, or any language. Maintains the same philosophy of maximum clarity with minimum words.

### Removed

**Agents removed (Ruby/Rails/Python-specific):**
- `dhh-rails-reviewer` - Rails-specific code review
- `kieran-rails-reviewer` - Rails-specific code review
- `kieran-python-reviewer` - Python-specific code review
- `lint` - Ruby/ERB linting (StandardRB, ERBLint, Brakeman)
- `julik-frontend-races-reviewer` - Stimulus/Hotwire-specific frontend review

**Skills removed (Ruby/Rails-specific):**
- `andrew-kane-gem-writer` - Ruby gem writing patterns
- `dhh-rails-style` - Rails code style
- `dhh-ruby-style` - Ruby code style
- `dspy-ruby` - Ruby LLM framework

### Migration Guide

If you're using this plugin with Ruby/Rails projects, you have two options:

1. **Stay on v2.16.0** - Pin your plugin version to continue using Ruby/Rails components
2. **Use generic agents** - The remaining review agents (architecture-strategist, security-sentinel, performance-oracle, etc.) still work for any language

## [2.16.0] - 2025-12-19

### Added

- **`david-react-reviewer` agent** - React code review with David's strict conventions.
- **`david-typescript-reviewer` agent** - TypeScript code review with David's strict type safety focus.

## [2.15.1] - 2025-12-18

### Changed

- **`/workflows:review` command** - Section 7 now detects project type (Web, iOS, or Hybrid) and offers appropriate testing. Web projects get `/playwright-test`, iOS projects get `/xcode-test`, hybrid projects can run both.

## [2.15.0] - 2025-12-18

### Added

- **`/xcode-test` command** - Build and test iOS apps on simulator using XcodeBuildMCP. Discovers projects/schemes, builds for simulator, installs and launches apps, takes screenshots, captures console logs, and supports human verification for Sign in with Apple, push notifications, and in-app purchases. Checks for XcodeBuildMCP installation first.

## [2.14.0] - 2025-12-18

### Added

- **`/playwright-test` command** - Run end-to-end browser tests on pages affected by a PR or branch. Uses Playwright MCP to navigate pages, capture snapshots, check console errors, test interactions, and pause for human verification on OAuth/email/payment flows. Creates P1 todos for failures and retries until passing.

### Changed

- **`/workflows:review` command** - Added optional Playwright testing phase (Section 7). After review agents complete, offers to spawn `/playwright-test` as a subagent to verify affected pages in a real browser.

## [2.13.0] - 2025-12-15

### Added

- **`dhh-rails-style` skill** - Write Ruby and Rails code in DHH's distinctive 37signals style. Router-pattern skill with sectioned references for controllers, models, frontend, architecture, and gems. Embodies REST purity, fat models, thin controllers, Current attributes, Hotwire patterns, and the "clarity over cleverness" philosophy. Based on analysis of Fizzy (Campfire) codebase.

## [2.12.0] - 2025-12-15

### Added

- **`data-migration-expert` agent** - New review agent for validating database migrations and data backfills. Ensures ID mappings match production reality, checks for swapped values, verifies rollback safety, and provides SQL verification snippets. Prevents silent data corruption from mismatched enum/ID mappings.
- **`deployment-verification-agent` agent** - New review agent that produces Go/No-Go deployment checklists for risky data changes. Creates pre/post-deploy SQL verification queries, defines data invariants, documents rollback procedures, and plans post-deploy monitoring.

### Changed

- **`/workflows:review` command** - Added conditional agents section. Now automatically runs `data-migration-expert` and `deployment-verification-agent` when PR contains database migrations (`db/migrate/*.rb`), data backfills, or ID/enum mapping changes.

## [2.11.0] - 2025-12-10

### Changed

- **Command naming convention** - Workflow commands now use `workflows:` prefix to avoid collisions with built-in Claude Code commands:
  - `/workflows:plan` (was `/plan`)
  - `/workflows:review` (was `/review`)
  - `/workflows:work` (was `/work`)
  - `/workflows:compound` (was `/compound`)

  This ensures no collision with Claude Code's built-in `/plan` command.

### Fixed

- **`heal-skill.md`** - Added missing `name:` frontmatter field
- **`create-agent-skill.md`** - Added missing `name:` frontmatter field
- **`prime.md`** - Rewrote corrupted command file (was incorrectly containing CLAUDE.md content)
- **Playwright MCP alias** - Shortened from `playwright` to `pw` to stay under 64-char API limit

### Removed

- **`prime.md`** - Removed from plugin (personal setup command, not for distribution)
- **`codify.md`** - Removed deprecated command (replaced by `/compound`)

## [2.10.0] - 2025-12-10

### Added

- **`agent-native-reviewer` agent** - New review agent that verifies features are agent-native. Checks that any action a user can take, an agent can also take (Action Parity), and anything a user can see, an agent can see (Context Parity). Enforces the principle: "Whatever the user can do, the agent can do."
- **`agent-native-architecture` skill** - Build AI agents using prompt-native architecture where features are defined in prompts, not code. Includes patterns for MCP tool design, system prompts, self-modification, and refactoring to prompt-native.

### Changed

- **`/review` command** - Added `agent-native-reviewer` to the parallel review agents. Code reviews now automatically check if new features are accessible to agents.

### Fixed

- **Documentation** - Fixed mermaid diagram legibility in dark mode by changing stroke color to white (PR #45 by @rickmanelius)

## [2.9.4] - 2025-12-08

### Changed

- **`/work` command** - Improved screenshot documentation for PR creation. Made capturing screenshots REQUIRED for any UI changes. Updated to use `imgup` skill with `pixhost` as default host (no API key needed). Clarified what to capture: new screens, before/after for modifications, and Figma design matches.

## [2.9.3] - 2025-12-05

### Changed

- **`/plan` command** - Added "Open plan in editor" as the first option in post-generation menu. This opens the plan file in the user's default editor for review before deciding on next steps.

## [2.9.2] - 2025-12-04

### Added

- **`/work` command** - Added screenshot documentation step for UI changes. Before creating PRs, use Playwright to capture before/after screenshots, upload via 0x0.st (using imgup skill), and include in PR description.

## [2.9.1] - 2025-12-04

### Changed

- **`/plan` command** - Reordered post-generation options: Review first, Work locally second, Work on remote third (using `&` for background execution). Removed "Rework" as separate option since "Other" handles custom changes.

## [2.9.0] - 2025-12-02

### Changed

- **Plugin renamed** from `compounding-engineering` to `compound-engineering`. Shorter name, same philosophy. Users will need to reinstall with the new name.

### Fixed

- **Documentation counts** - Updated all documentation to reflect actual component counts (24 agents, 19 commands).

## [2.8.3] - 2025-11-29

### Fixed

- **`gemini-imagegen` skill** - Added critical documentation about file format handling. Gemini returns JPEG by default, so using `.jpg` extension is required to avoid "Image does not match media type" API errors. Added examples for PNG conversion when needed and format verification.

## [2.8.2] - 2025-11-28

### Changed

- **`gemini-imagegen` skill** - Updated to use only Pro model (`gemini-2.0-flash-preview-image-generation`) by default. Removed regular Nano Banana model reference. Added explicit options for aspect ratio (1:1 to 21:9) and resolution (1K default, 2K, 4K). Simplified documentation with clear defaults.

## [2.8.1] - 2025-11-27

### Added

- **`/plan` command** - Added "Create Issue" option to post-generation menu. Detects project tracker (GitHub or Linear) from user's CLAUDE.md (`project_tracker: github` or `project_tracker: linear`) and creates issues using `gh issue create` or Linear CLI.

## [2.8.0] - 2025-11-27

### Added

- **`julik-frontend-races-reviewer` agent** - New review agent specializing in JavaScript and Stimulus code race conditions. Reviews frontend code with Julik's eye for timing issues, DOM event handling, promise management, setTimeout/setInterval cleanup, CSS animations, and concurrent operation tracking. Includes patterns for Hotwire/Turbo compatibility and state machine recommendations.

## [2.7.0] - 2025-11-27

### Changed

- **`/codify` → `/compound`** - Renamed the documentation command to better reflect the compounding engineering philosophy. Each documented solution compounds your team's knowledge. The old `/codify` command still works but shows a deprecation notice and calls `/compound`.
- **`codify-docs` → `compound-docs`** - Renamed the skill to match the new command name.

### Updated

- All documentation, philosophy sections, and references updated to use `/compound` and `compound-docs`

## [2.6.2] - 2025-11-27

### Improved

- **`/plan` command** - Added AskUserQuestion tool for post-generation options and year note (2025) for accurate date awareness.
- **Research agents** - Added year note (2025) to all 4 research agents (best-practices-researcher, framework-docs-researcher, git-history-analyzer, repo-research-analyst) for accurate date awareness when searching documentation.

## [2.6.1] - 2025-11-26

### Improved

- **`/plan` command** - Replaced vague "keep asking questions" ending with clear post-generation options menu. Users now see 4 explicit choices via AskUserQuestion: Start `/work`, Run `/plan_review`, Simplify, or Rework.

## [2.6.0] - 2024-11-26

### Removed

- **`feedback-codifier` agent** - Removed from workflow agents. Agent count reduced from 24 to 23.

## [2.5.0] - 2024-11-25

### Added

- **`/report-bug` command** - New slash command for reporting bugs in the compound-engineering plugin. Provides a structured workflow that gathers bug information through guided questions, collects environment details automatically, and creates a GitHub issue in the EveryInc/every-marketplace repository. Designed to be user-friendly for anyone using the plugin.

## [2.4.1] - 2024-11-24

### Improved

- **design-iterator agent** - Added focused screenshot guidance: always capture only the target element/area instead of full page screenshots. Includes browser_resize recommendations, element-targeted screenshot workflow using browser_snapshot refs, and explicit instruction to never use fullPage mode. Also added step to load relevant design skills (e.g., Swiss design) before starting iterations.

## [2.4.0] - 2024-11-24

### Fixed

- **MCP Configuration** - Moved MCP servers back to `plugin.json` following working examples from anthropics/life-sciences plugins. Removed `.mcp.json` file as it's not the correct approach.
- **Context7 URL** - Updated to use HTTP type with correct endpoint URL.

## [2.3.0] - 2024-11-24

### Changed

- **MCP Configuration** - Moved MCP servers from inline `plugin.json` to separate `.mcp.json` file per Claude Code best practices for plugin MCP integration.

## [2.2.1] - 2024-11-24

### Fixed

- **Playwright MCP Server** - Added missing `"type": "stdio"` field required for MCP server configuration to load properly.

## [2.2.0] - 2024-11-24

### Added

- **Context7 MCP Server** - Bundled Context7 for instant framework documentation lookup. Provides up-to-date docs for Rails, React, Next.js, and 100+ other frameworks.

## [2.1.0] - 2024-11-24

### Added

- **Playwright MCP Server** - Bundled `@playwright/mcp` for browser automation across all projects using this plugin. Provides screenshot, navigation, click, fill, and evaluate tools.

### Changed

- Replaced all Puppeteer references with Playwright across agents and commands:
  - `bug-reproduction-validator` agent
  - `design-iterator` agent
  - `design-implementation-reviewer` agent
  - `figma-design-sync` agent
  - `generate_command` command

## [2.0.2] - 2024-11-24

### Changed

- `design-iterator` agent - Updated description to emphasize proactive usage when design work isn't coming together on first attempt. Added examples showing how to suggest 5x or 10x iterations when initial changes don't fully resolve design issues.

## [2.0.1] - 2024-11-24

### Added

- `CLAUDE.md` - Project instructions with versioning requirements and pre-commit checklist
- `docs/solutions/plugin-versioning-requirements.md` - Detailed workflow documentation for maintaining version, CHANGELOG, and README in sync

## [2.0.0] - 2024-11-24

Major reorganization consolidating agents, commands, and skills from multiple sources into a single, well-organized plugin.

### Added

**New Agents (7)**
- `design-iterator` - Iteratively refine UI components through systematic design iterations
- `design-implementation-reviewer` - Verify UI implementations match Figma design specifications
- `figma-design-sync` - Synchronize web implementations with Figma designs
- `bug-reproduction-validator` - Systematically reproduce and validate bug reports
- `spec-flow-analyzer` - Analyze user flows and identify gaps in specifications
- `lint` - Run linting and code quality checks on Ruby and ERB files
- `ankane-readme-writer` - Create READMEs following Ankane-style template for Ruby gems

**New Commands (9)**
- `/changelog` - Create engaging changelogs for recent merges
- `/plan_review` - Multi-agent plan review in parallel
- `/resolve_parallel` - Resolve TODO comments in parallel
- `/resolve_pr_parallel` - Resolve PR comments in parallel
- `/reproduce-bug` - Reproduce bugs using logs and console
- `/prime` - Prime/setup command
- `/create-agent-skill` - Create or edit Claude Code skills
- `/heal-skill` - Fix skill documentation issues
- `/codify` - Document solved problems for knowledge base

**New Skills (10)**
- `andrew-kane-gem-writer` - Write Ruby gems following Andrew Kane's patterns
- `codify-docs` - Capture solved problems as categorized documentation
- `create-agent-skills` - Expert guidance for creating Claude Code skills
- `dhh-ruby-style` - Write Ruby/Rails code in DHH's 37signals style
- `dspy-ruby` - Build type-safe LLM applications with DSPy.rb
- `every-style-editor` - Review copy for Every's style guide compliance
- `file-todos` - File-based todo tracking system
- `frontend-design` - Create production-grade frontend interfaces
- `git-worktree` - Manage Git worktrees for parallel development
- `skill-creator` - Guide for creating effective Claude Code skills

### Changed

**Agents Reorganized by Category**
- `review/` (10 agents) - Code quality, security, performance reviewers
- `research/` (4 agents) - Documentation, patterns, history analysis
- `design/` (3 agents) - UI/design review and iteration
- `workflow/` (6 agents) - PR resolution, bug validation, linting
- `docs/` (1 agent) - README generation

**Commands Restructured**
- Workflow commands moved to `commands/workflows/` subdirectory
- `/plan`, `/review`, `/work`, `/codify` accessible via short names (autocomplete) or full path

### Summary

| Component | v1.1.0 | v2.0.0 | Change |
|-----------|--------|--------|--------|
| Agents | 17 | 24 | +7 |
| Commands | 6 | 15 | +9 |
| Skills | 1 | 11 | +10 |

---

## [1.1.0] - 2024-11-22

### Added

**gemini-imagegen Skill**
- Text-to-image generation with Google's Gemini API
- Image editing and manipulation
- Multi-turn refinement via chat interface
- Multiple reference image composition (up to 14 images)
- Model support: `gemini-2.5-flash-image` and `gemini-3-pro-image-preview`
- Python scripts: `generate_image.py`, `edit_image.py`, `multi_turn_chat.py`, `compose_images.py`

### Fixed
- Corrected component counts in documentation (17 agents, not 15)

### Documentation
- Added comprehensive README with all components listed
- Added this changelog
- Added `requirements.txt` for Python dependencies

---

## [1.0.0] - 2024-10-09

Initial release of the compound-engineering plugin.

### Added

**17 Specialized Agents**

*Code Review (5)*
- `kieran-rails-reviewer` - Rails code review with strict conventions
- `kieran-python-reviewer` - Python code review with quality standards
- `kieran-typescript-reviewer` - TypeScript code review
- `dhh-rails-reviewer` - Rails review from DHH's perspective
- `code-simplicity-reviewer` - Final pass for simplicity and minimalism

*Analysis & Architecture (4)*
- `architecture-strategist` - Architectural decisions and compliance
- `pattern-recognition-specialist` - Design pattern analysis
- `security-sentinel` - Security audits and vulnerability assessments
- `performance-oracle` - Performance analysis and optimization
- `data-integrity-guardian` - Database migrations and data integrity

*Research (4)*
- `framework-docs-researcher` - Framework documentation research
- `best-practices-researcher` - External best practices gathering
- `git-history-analyzer` - Git history and code evolution analysis
- `repo-research-analyst` - Repository structure and conventions

*Workflow (3)*
- `every-style-editor` - Every's style guide compliance
- `pr-comment-resolver` - PR comment resolution
- `feedback-codifier` - Feedback pattern codification

**6 Slash Commands**
- `/plan` - Create implementation plans
- `/review` - Comprehensive code reviews
- `/work` - Execute work items systematically
- `/triage` - Triage and prioritize issues
- `/resolve_todo_parallel` - Resolve TODOs in parallel
- `/generate_command` - Generate new slash commands

**Infrastructure**
- MIT license
- Plugin manifest (`plugin.json`)
- Pre-configured permissions for Rails development
