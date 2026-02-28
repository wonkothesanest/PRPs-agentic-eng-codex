# Feature: PRP Framework Codex Port — Phase 1 (Scaffold + Phase 2 Skills + Phase 3 Agents)

## Summary

Convert the PRP framework's Claude Code commands and agent definitions into OpenAI Codex-compatible SKILL.md files, create `AGENTS.md` from `CLAUDE.md`, and build an install README. The result is a parallel `.agents/skills/` tree at repo root that Codex auto-discovers alongside the existing `.claude/` tree, leaving Claude Code compatibility untouched.

This plan covers **all 5 phases** since Phase 1 (scaffold), Phase 2 (skills), and Phase 3 (agents) are small pure-translation tasks. Phase 4 (live Codex validation) is manual; Phase 5 (README) follows from it.

## User Story

As a developer using OpenAI Codex
I want to invoke `$prp-plan`, `$prp-implement`, `$prp-commit`, and other PRP skills
So that I can use the full PRP methodology without switching to Claude Code

## Problem Statement

11 command files and 11 agent definition files exist only as Claude Code artifacts. Converting them requires: (1) adding SKILL.md frontmatter, (2) replacing Claude Code-specific tool references, (3) creating `AGENTS.md`.

## Solution Statement

Additive file creation only. No existing files modified. New tree at `.agents/skills/` mirrors the PRP command set as SKILL.md files. Agents become sub-skills in `.agents/skills/`. `AGENTS.md` is `CLAUDE.md` with Codex-specific invocation references.

## Metadata

| Field | Value |
|-------|-------|
| Type | NEW_CAPABILITY |
| Complexity | LOW |
| Systems Affected | `.agents/skills/`, `AGENTS.md`, `CODEX.md` (install README) |
| Dependencies | None external |
| Estimated Tasks | 27 |

---

## UX Design

### Before State

```
╔══════════════════════════════════════════════════════════╗
║  Developer opens Codex session                           ║
║  Types: $prp-plan                                        ║
║  Result: "Unknown skill"                                 ║
║                                                          ║
║  Only option: manually copy/paste command body           ║
╚══════════════════════════════════════════════════════════╝
```

### After State

```
╔══════════════════════════════════════════════════════════╗
║  Developer clones repo, opens Codex in project dir       ║
║  Types: $prp-plan implement user authentication          ║
║  Result: Full PRP plan created in .claude/PRPs/plans/    ║
║                                                          ║
║  All 11 skills available in Codex $-browser              ║
╚══════════════════════════════════════════════════════════╝
```

### Interaction Changes

| Location | Before | After | User Impact |
|----------|--------|-------|-------------|
| Codex session | No PRP skills | 11 skills loadable | Full PRP workflow available |
| Project context | No AGENTS.md | AGENTS.md present | Codex reads PRP methodology on startup |
| Install | Nothing | Clone + README | 3-step onboarding |

---

## Mandatory Reading

**CRITICAL: Implementation agent MUST read these files before starting:**

| Priority | File | Lines | Why Read This |
|----------|------|-------|---------------|
| P0 | `.claude/skills/prp-core-runner/SKILL.md` | all | EXACT frontmatter + body pattern to mirror |
| P0 | `.claude/agents/codebase-explorer.md` | 1-6 | Agent frontmatter to translate to SKILL.md |
| P0 | `.claude/agents/web-researcher.md` | 1-6 | Second agent frontmatter example |
| P1 | `.claude/commands/prp-core/prp-commit.md` | all | Shortest command - port this one first |
| P1 | `.claude/commands/prp-core/prp-plan.md` | all | Most complex command (agents + $ARGUMENTS) |
| P1 | `CLAUDE.md` | all | Source for AGENTS.md adaptation |
| P2 | `.claude/commands/prp-core/prp-implement.md` | 50-80 | Shows $ARGUMENTS used in bash commands |

**External Documentation:**

| Source | Section | Why Needed |
|--------|---------|------------|
| [agentskills.io/specification](https://agentskills.io/specification) | Frontmatter fields | Authoritative SKILL.md format |
| [github.com/openai/skills curated](https://github.com/openai/skills/tree/main/skills/.curated) | Any skill body | Real-world SKILL.md body patterns |
| [developers.openai.com/codex/skills/](https://developers.openai.com/codex/skills/) | Discovery paths | Confirms `.agents/skills/` auto-discovery |

---

## Patterns to Mirror

**SKILL.MD_FRONTMATTER (from `.claude/skills/prp-core-runner/SKILL.md:1-4`):**

```yaml
---
name: prp-core-runner
description: Orchestrate complete PRP workflow from feature request to pull request. Run create branch, create PRP, execute implementation, commit changes, and create PR in sequence. Use when implementing features using PRP methodology or when user requests full PRP workflow.
---
```

Key rules:
- `name`: kebab-case, matches directory name, matches source command filename without `.md`
- `description`: Include an explicit "Use when..." trigger condition. Take from command's `description` frontmatter field.

**COMMAND_SOURCE_FRONTMATTER (from `.claude/commands/prp-core/prp-commit.md:1-4`):**

```yaml
---
description: Quick commit with natural language file targeting
argument-hint: [target description] (blank = all changes)
---
```

The `description` field becomes SKILL.md's `description`. The `argument-hint` field is discarded (no equivalent). The slash command name (e.g., `prp-commit`) becomes the SKILL.md `name`.

**ARGUMENTS_REPLACEMENT:**

Original in commands:
```
**Target**: $ARGUMENTS
```

Replace with:
```
**Target**: The user's target description from their request (or all changes if not specified)
```

General rule: `$ARGUMENTS` → phrase describing what the user provides in natural language. Each command's `argument-hint` explains what it expects - use that to write the replacement phrase.

**AGENT_SPAWNING_REPLACEMENT:**

Original in commands:
```
Use Task tool with `subagent_type="prp-core:codebase-explorer"`:
```

Replace with:
```
Use the `$codebase-explorer` skill with the following prompt:
```

General rule: `Task tool with subagent_type="prp-core:X"` → `Use the \`$X\` skill`

**SLASHCOMMAND_REPLACEMENT (from `.claude/skills/prp-core-runner/SKILL.md:10`):**

Original:
```
use the SlashCommand tool to invoke `/prp-core-run-all`
```

Replace with:
```
invoke the `$prp-core-runner` skill
```

**AGENT_SKILL_FRONTMATTER (from `.claude/agents/codebase-explorer.md:1-6`):**

```yaml
---
name: codebase-explorer
description: Comprehensive codebase exploration - finds WHERE code lives AND shows HOW it's implemented. Use when you need to locate files, understand directory structure, AND extract actual code patterns. Combines file finding with pattern extraction in one pass.
model: sonnet
color: green
---
```

→ Becomes SKILL.md frontmatter (drop `model` and `color`, keep `name` and `description`):

```yaml
---
name: codebase-explorer
description: Comprehensive codebase exploration - finds WHERE code lives AND shows HOW it's implemented. Use when you need to locate files, understand directory structure, AND extract actual code patterns. Combines file finding with pattern extraction in one pass.
---
```

Agent body content (everything after the frontmatter `---`) is kept verbatim.

---

## Files to Create

| File | Action | Source |
|------|--------|--------|
| `AGENTS.md` | CREATE | Adapt from `CLAUDE.md` |
| `.agents/skills/prp-plan/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-plan.md` |
| `.agents/skills/prp-implement/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-implement.md` |
| `.agents/skills/prp-commit/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-commit.md` |
| `.agents/skills/prp-pr/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-pr.md` |
| `.agents/skills/prp-review/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-review.md` |
| `.agents/skills/prp-review-agents/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-review-agents.md` |
| `.agents/skills/prp-debug/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-debug.md` |
| `.agents/skills/prp-prd/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-prd.md` |
| `.agents/skills/prp-codebase-question/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-codebase-question.md` |
| `.agents/skills/prp-issue-investigate/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-issue-investigate.md` |
| `.agents/skills/prp-issue-fix/SKILL.md` | CREATE | `.claude/commands/prp-core/prp-issue-fix.md` |
| `.agents/skills/prp-core-runner/SKILL.md` | CREATE | `.claude/skills/prp-core-runner/SKILL.md` (update) |
| `.agents/skills/codebase-explorer/SKILL.md` | CREATE | `.claude/agents/codebase-explorer.md` |
| `.agents/skills/codebase-analyst/SKILL.md` | CREATE | `.claude/agents/codebase-analyst.md` |
| `.agents/skills/web-researcher/SKILL.md` | CREATE | `.claude/agents/web-researcher.md` |
| `.agents/skills/code-reviewer/SKILL.md` | CREATE | `.claude/agents/code-reviewer.md` |
| `.agents/skills/code-simplifier/SKILL.md` | CREATE | `.claude/agents/code-simplifier.md` |
| `.agents/skills/comment-analyzer/SKILL.md` | CREATE | `.claude/agents/comment-analyzer.md` |
| `.agents/skills/docs-impact-agent/SKILL.md` | CREATE | `.claude/agents/docs-impact-agent.md` |
| `.agents/skills/pr-test-analyzer/SKILL.md` | CREATE | `.claude/agents/pr-test-analyzer.md` |
| `.agents/skills/silent-failure-hunter/SKILL.md` | CREATE | `.claude/agents/silent-failure-hunter.md` |
| `.agents/skills/type-design-analyzer/SKILL.md` | CREATE | `.claude/agents/type-design-analyzer.md` |
| `.agents/skills/gpui-researcher/SKILL.md` | CREATE | `.claude/agents/gpui-researcher.md` |
| `CODEX.md` | CREATE | Installation + usage README for Codex |

**Files NOT modified:** anything in `.claude/` (existing Claude Code structure stays untouched)

---

## NOT Building (Scope Limits)

- `prp-ralph` and `prp-ralph-cancel` - lifecycle hooks not available in Codex
- `agents/openai.yaml` per-skill config - no MCP dependencies, not needed
- SkillsMP marketplace submission - out of scope
- `config.toml` - no project-specific Codex config needed
- Any CI/CD or automated testing

---

## Step-by-Step Tasks

Execute in order. Each task is atomic and independently verifiable.

---

### Task 1: CREATE directory scaffolding

- **ACTION**: Create all required directories in one pass
- **RUN**:
  ```bash
  mkdir -p .agents/skills/prp-plan
  mkdir -p .agents/skills/prp-implement
  mkdir -p .agents/skills/prp-commit
  mkdir -p .agents/skills/prp-pr
  mkdir -p .agents/skills/prp-review
  mkdir -p .agents/skills/prp-review-agents
  mkdir -p .agents/skills/prp-debug
  mkdir -p .agents/skills/prp-prd
  mkdir -p .agents/skills/prp-codebase-question
  mkdir -p .agents/skills/prp-issue-investigate
  mkdir -p .agents/skills/prp-issue-fix
  mkdir -p .agents/skills/prp-core-runner
  mkdir -p .agents/skills/codebase-explorer
  mkdir -p .agents/skills/codebase-analyst
  mkdir -p .agents/skills/web-researcher
  mkdir -p .agents/skills/code-reviewer
  mkdir -p .agents/skills/code-simplifier
  mkdir -p .agents/skills/comment-analyzer
  mkdir -p .agents/skills/docs-impact-agent
  mkdir -p .agents/skills/pr-test-analyzer
  mkdir -p .agents/skills/silent-failure-hunter
  mkdir -p .agents/skills/type-design-analyzer
  mkdir -p .agents/skills/gpui-researcher
  ```
- **VALIDATE**: `ls .agents/skills/ | wc -l` should output `23`

---

### Task 2: CREATE `AGENTS.md`

- **ACTION**: Create `AGENTS.md` by adapting `CLAUDE.md`
- **SOURCE**: Read `CLAUDE.md` fully, then write `AGENTS.md` with these changes:
  1. Change title line from `# CLAUDE.md` to `# AGENTS.md`
  2. Change opening sentence from "This file provides guidance to Claude Code..." to "This file provides guidance to OpenAI Codex..."
  3. In `## Core Architecture > ### Command-Driven System`: add a note that Codex users use `.agents/skills/` instead of `.claude/commands/`
  4. In `## Development Commands > ### Key Claude Commands`: change section heading to `### Key Codex Skills` and prefix all command references with `$` instead of `/` (e.g., `/prp-commit` → `$prp-commit`). Remove the `prp-core-run-all` reference (not ported). Remove `prp-ralph` and `prp-ralph-cancel`.
  5. In `## Working with This Framework > ### Command Usage`: replace "Access via `/` prefix in Claude Code" with "Access via `$` prefix in Codex"
  6. Everything else: copy verbatim
- **GOTCHA**: Do not remove the PRP methodology sections - they are just as relevant for Codex
- **VALIDATE**: `test -f AGENTS.md && echo "OK"`

---

### Task 3: CREATE `.agents/skills/prp-commit/SKILL.md`

Start with the simplest command to establish the pattern.

- **ACTION**: Read `.claude/commands/prp-core/prp-commit.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-commit
  description: Quick commit with natural language file targeting. Use when the developer wants to stage and commit changes by describing what to commit rather than specifying file paths explicitly.
  ---
  ```
- **BODY**: Copy full command body verbatim with these substitutions:
  1. `**Target**: $ARGUMENTS` → `**Target**: The developer's description of what to commit (from their request), or all changes if not specified`
  2. No other `$ARGUMENTS` occurrences in this file
- **VALIDATE**: `head -5 .agents/skills/prp-commit/SKILL.md` should show the `---` frontmatter block

---

### Task 4: CREATE `.agents/skills/prp-pr/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-pr.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-pr
  description: Create a PR from the current branch with unpushed commits. Use when the developer is ready to push their branch and open a pull request. Accepts an optional base branch (defaults to main).
  ---
  ```
- **BODY**: Copy verbatim, replace:
  - `**Base branch**: $ARGUMENTS (default: main)` → `**Base branch**: The base branch from the developer's request, or main if not specified`
- **VALIDATE**: `head -5 .agents/skills/prp-pr/SKILL.md`

---

### Task 5: CREATE `.agents/skills/prp-review/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-review.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-review
  description: Comprehensive PR code review acting as a senior engineer. Use when a developer wants a high-quality single-pass review of a pull request or set of code changes. Accepts an optional PR number or diff target.
  ---
  ```
- **BODY**: Copy verbatim, replace:
  - `**Input**: $ARGUMENTS` → `**Input**: The PR number, branch, or diff target from the developer's request (or review the current branch's diff if not specified)`
- **VALIDATE**: `head -5 .agents/skills/prp-review/SKILL.md`

---

### Task 6: CREATE `.agents/skills/prp-debug/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-debug.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-debug
  description: Deep root cause analysis for bugs and errors - finds the actual cause, not just symptoms. Use when a developer needs to investigate a bug, error, or unexpected behavior using systematic 5-Why analysis.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. `**Input**: $ARGUMENTS` → `**Input**: The bug description, error message, or problem statement from the developer's request`
  2. `Use Task tool with \`subagent_type="prp-core:codebase-analyst"\`:` → `Use the \`$codebase-analyst\` skill with the following prompt:`
- **VALIDATE**: `head -5 .agents/skills/prp-debug/SKILL.md`

---

### Task 7: CREATE `.agents/skills/prp-codebase-question/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-codebase-question.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-codebase-question
  description: Research codebase questions using parallel specialized agents - documents what exists, not what should change. Use when a developer has a question about how the codebase works, where code lives, or how a feature is implemented.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. `**Input**: $ARGUMENTS` → `**Input**: The codebase question from the developer's request`
  2. All occurrences of `Use Task tool with \`subagent_type="prp-core:codebase-explorer"\`:` → `Use the \`$codebase-explorer\` skill with the following prompt:`
  3. All occurrences of `Use Task tool with \`subagent_type="prp-core:codebase-analyst"\`:` → `Use the \`$codebase-analyst\` skill with the following prompt:`
  4. All occurrences of `Use Task tool with \`subagent_type="prp-core:web-researcher"\`:` → `Use the \`$web-researcher\` skill with the following prompt:`
  5. Any inline `\`prp-core:X\`` agent name references in prose → `\`$X\``
- **VALIDATE**: `grep -c 'subagent_type' .agents/skills/prp-codebase-question/SKILL.md` should output `0`

---

### Task 8: CREATE `.agents/skills/prp-issue-investigate/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-issue-investigate.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-issue-investigate
  description: Investigate a GitHub issue or bug report - analyzes the codebase, creates an investigation plan, and optionally posts findings to GitHub. Use when a developer wants to deeply investigate an issue before fixing it.
  ---
  ```
- **BODY**: Copy verbatim with same agent-spawning substitutions as Task 7
- **VALIDATE**: `grep -c 'subagent_type' .agents/skills/prp-issue-investigate/SKILL.md` should output `0`

---

### Task 9: CREATE `.agents/skills/prp-issue-fix/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-issue-fix.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-issue-fix
  description: Implement a fix from an investigation artifact - applies code changes, creates a PR, and performs self-review. Use when a developer has an investigation plan ready and wants to execute the fix.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. `**Input**: $ARGUMENTS` → `**Input**: The investigation artifact path or fix description from the developer's request`
  2. `Use Task tool with subagent_type="code-reviewer":` → `Use the \`$code-reviewer\` skill with the following prompt:`
- **VALIDATE**: `grep -c 'subagent_type' .agents/skills/prp-issue-fix/SKILL.md` should output `0`

---

### Task 10: CREATE `.agents/skills/prp-review-agents/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-review-agents.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-review-agents
  description: Comprehensive PR review using 7 specialized agents covering comments, tests, errors, types, code quality, docs, and simplification. Use when a developer wants a thorough multi-perspective review of a PR.
  ---
  ```
- **BODY**: Copy verbatim. This command references agents in prose (not via `subagent_type=` syntax) - update prose references:
  - Any "via Task tool" reference → "by invoking the `$X` skill"
  - Agent names in prose stay as-is (e.g., `code-reviewer`, `docs-impact-agent`) but add `$` prefix where they describe invocations: → `$code-reviewer`, `$docs-impact-agent`, etc.
- **VALIDATE**: `head -5 .agents/skills/prp-review-agents/SKILL.md`

---

### Task 11: CREATE `.agents/skills/prp-prd/SKILL.md`

- **ACTION**: Read `.claude/commands/prp-core/prp-prd.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-prd
  description: Interactive PRD generator using a problem-first, hypothesis-driven approach. Use when a developer or product manager wants to create a structured Product Requirements Document through a guided conversation.
  ---
  ```
- **BODY**: Copy verbatim with same agent-spawning substitutions as Task 7 (this command uses web-researcher, codebase-explorer, codebase-analyst)
- **VALIDATE**: `grep -c 'subagent_type' .agents/skills/prp-prd/SKILL.md` should output `0`

---

### Task 12: CREATE `.agents/skills/prp-plan/SKILL.md`

Most complex command. Read carefully before creating.

- **ACTION**: Read `.claude/commands/prp-core/prp-plan.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-plan
  description: Create a comprehensive feature implementation plan with deep codebase analysis, pattern extraction, and strategic research. Use when a developer wants to plan implementation of a feature, bug fix, or refactor before writing any code.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. Inside the `<objective>` tag: `Transform "$ARGUMENTS" into a battle-tested implementation plan` → `Transform the developer's feature request into a battle-tested implementation plan`
  2. All agent-spawning substitutions (Task tool → `$skill-name`)
  3. Keep all XML-style tags (`<objective>`, `<context>`, `<process>`, etc.) verbatim - they are content instructions, not Claude Code syntax
- **GOTCHA**: `prp-plan.md` uses `$ARGUMENTS` inside both XML tags AND inside bash-like references - check every occurrence
- **VALIDATE**: `grep -c '\$ARGUMENTS' .agents/skills/prp-plan/SKILL.md` should output `0`
- **VALIDATE**: `grep -c 'subagent_type' .agents/skills/prp-plan/SKILL.md` should output `0`

---

### Task 13: CREATE `.agents/skills/prp-implement/SKILL.md`

Complex - uses `$ARGUMENTS` in bash commands.

- **ACTION**: Read `.claude/commands/prp-core/prp-implement.md` fully, then create the skill
- **FRONTMATTER**:
  ```yaml
  ---
  name: prp-implement
  description: Execute a feature implementation plan with rigorous validation loops - reads the plan, implements each task, runs validations, and iterates until all checks pass. Use when a developer has a .plan.md file ready and wants to execute it.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. `**Plan**: $ARGUMENTS` → `**Plan**: The plan file path from the developer's request`
  2. `cat $ARGUMENTS` → `cat {the plan file path from the developer's request}`
  3. `Error: Plan not found at $ARGUMENTS` → `Error: Plan not found at the specified path`
  4. All bash references to `$ARGUMENTS` as a file path → `{plan_file_path}` with a note that the AI should use the path from the user's request
  5. `mv $ARGUMENTS .claude/PRPs/plans/completed/` → `mv {plan_file_path} .claude/PRPs/plans/completed/` (keep the destination path - Codex can read/write files)
- **GOTCHA**: `$ARGUMENTS` appears 7 times in `prp-implement.md` - replace every one
- **VALIDATE**: `grep -c '\$ARGUMENTS' .agents/skills/prp-implement/SKILL.md` should output `0`

---

### Task 14: CREATE `.agents/skills/prp-core-runner/SKILL.md`

- **ACTION**: Read `.claude/skills/prp-core-runner/SKILL.md` fully, then create the new version
- **FRONTMATTER**: Keep verbatim (already correct SKILL.md format):
  ```yaml
  ---
  name: prp-core-runner
  description: Orchestrate complete PRP workflow from feature request to pull request. Run create branch, create PRP, execute implementation, commit changes, and create PR in sequence. Use when implementing features using PRP methodology or when user requests full PRP workflow.
  ---
  ```
- **BODY**: Copy verbatim with these substitutions:
  1. `use the SlashCommand tool to invoke \`/prp-core-run-all\`` → `invoke the \`$prp-plan\` skill followed by \`$prp-implement\`, \`$prp-commit\`, and \`$prp-pr\` skills in sequence`
  2. `Use SlashCommand tool with \`/prp-core-run-all {feature-description}\`` → `Invoke \`$prp-plan {feature-description}\`, then \`$prp-implement\`, then \`$prp-commit\`, then \`$prp-pr\` in sequence`
  3. Update the step-by-step list to name individual skills instead of the single `prp-core-run-all` command
- **VALIDATE**: `grep -c 'SlashCommand' .agents/skills/prp-core-runner/SKILL.md` should output `0`

---

### Tasks 15–24: CREATE agent SKILL.md files

For each of the 11 agent files, create a corresponding SKILL.md. The pattern is identical for all:

**Pattern**: Read source agent `.md` file → Create `.agents/skills/{name}/SKILL.md`
- **FRONTMATTER**: Copy `name` and `description` from agent file. DROP `model` and `color`.
- **BODY**: Copy everything after the frontmatter `---` closing line verbatim. No substitutions needed (agent bodies contain no `$ARGUMENTS` or Claude Code tool references).

| Task | Source | Destination |
|------|--------|-------------|
| 15 | `.claude/agents/codebase-explorer.md` | `.agents/skills/codebase-explorer/SKILL.md` |
| 16 | `.claude/agents/codebase-analyst.md` | `.agents/skills/codebase-analyst/SKILL.md` |
| 17 | `.claude/agents/web-researcher.md` | `.agents/skills/web-researcher/SKILL.md` |
| 18 | `.claude/agents/code-reviewer.md` | `.agents/skills/code-reviewer/SKILL.md` |
| 19 | `.claude/agents/code-simplifier.md` | `.agents/skills/code-simplifier/SKILL.md` |
| 20 | `.claude/agents/comment-analyzer.md` | `.agents/skills/comment-analyzer/SKILL.md` |
| 21 | `.claude/agents/docs-impact-agent.md` | `.agents/skills/docs-impact-agent/SKILL.md` |
| 22 | `.claude/agents/pr-test-analyzer.md` | `.agents/skills/pr-test-analyzer/SKILL.md` |
| 23 | `.claude/agents/silent-failure-hunter.md` | `.agents/skills/silent-failure-hunter/SKILL.md` |
| 24 | `.claude/agents/type-design-analyzer.md` | `.agents/skills/type-design-analyzer/SKILL.md` |
| 25 | `.claude/agents/gpui-researcher.md` | `.agents/skills/gpui-researcher/SKILL.md` |

**VALIDATE each**: `head -5 .agents/skills/{name}/SKILL.md` should show `---` frontmatter with `name:` and `description:` but NOT `model:` or `color:`

---

### Task 26: VERIFY all SKILL.md files are clean

- **ACTION**: Run validation checks across all created skills
- **VALIDATE**:
  ```bash
  # No $ARGUMENTS remaining in any skill
  grep -r '\$ARGUMENTS' .agents/skills/ && echo "FAIL: $ARGUMENTS found" || echo "PASS: No $ARGUMENTS"

  # No subagent_type references remaining
  grep -r 'subagent_type' .agents/skills/ && echo "FAIL: subagent_type found" || echo "PASS: No subagent_type"

  # No SlashCommand tool references
  grep -r 'SlashCommand tool' .agents/skills/ && echo "FAIL: SlashCommand found" || echo "PASS: No SlashCommand"

  # All skill directories have a SKILL.md
  for dir in .agents/skills/*/; do
    if [ ! -f "${dir}SKILL.md" ]; then echo "MISSING: ${dir}SKILL.md"; fi
  done

  # All SKILL.md files have name: frontmatter
  for f in .agents/skills/*/SKILL.md; do
    if ! grep -q '^name:' "$f"; then echo "MISSING name: in $f"; fi
  done

  # Count total skills
  ls .agents/skills/ | wc -l  # should be 23 (11 commands + 12 agents)
  ```
- **EXPECT**: All PASS, no MISSING, count = 23

---

### Task 27: CREATE `CODEX.md` (Installation README)

- **ACTION**: Create a developer-facing installation and usage guide
- **CONTENT**:

```markdown
# Using the PRP Framework with OpenAI Codex

## Installation

1. Clone this repository (or add it as a git submodule):
   ```bash
   git clone https://github.com/your-org/PRPs-agentic-eng .prp-framework
   ```

2. Symlink (or copy) the skills directory into your project:
   ```bash
   ln -s /path/to/PRPs-agentic-eng/.agents/skills .agents/skills
   ```
   Or copy for isolated use:
   ```bash
   cp -r /path/to/PRPs-agentic-eng/.agents/skills .agents/skills
   ```

3. Copy `AGENTS.md` to your project root:
   ```bash
   cp /path/to/PRPs-agentic-eng/AGENTS.md AGENTS.md
   ```

## Available Skills

Invoke via `$skill-name` in Codex, or use `/skills` to browse.

### Core Workflow

| Skill | Description |
|-------|-------------|
| `$prp-plan` | Create implementation plan from feature description |
| `$prp-implement` | Execute a .plan.md file with validation loops |
| `$prp-commit` | Commit changes using natural language targeting |
| `$prp-pr` | Create PR from current branch |
| `$prp-core-runner` | Full workflow: plan → implement → commit → PR |

### Research & Debug

| Skill | Description |
|-------|-------------|
| `$prp-prd` | Generate a PRD through guided conversation |
| `$prp-codebase-question` | Research how the codebase works |
| `$prp-debug` | Deep root cause analysis |
| `$prp-issue-investigate` | Investigate a GitHub issue |
| `$prp-issue-fix` | Implement a fix from an investigation |

### Review

| Skill | Description |
|-------|-------------|
| `$prp-review` | Senior engineer PR review |
| `$prp-review-agents` | Multi-agent 7-aspect review |

### Specialized Agents (used by skills above)

`$codebase-explorer`, `$codebase-analyst`, `$web-researcher`, `$code-reviewer`,
`$code-simplifier`, `$comment-analyzer`, `$docs-impact-agent`, `$pr-test-analyzer`,
`$silent-failure-hunter`, `$type-design-analyzer`, `$gpui-researcher`

## Not Available in Codex

- `prp-ralph` / `prp-ralph-cancel` — requires lifecycle hooks not yet in Codex production

## Claude Code Users

The `.claude/` directory and all existing commands remain unchanged. This is additive only.
```

- **VALIDATE**: `test -f CODEX.md && wc -l CODEX.md`

---

## Testing Strategy

### Validation Checklist (manual Codex session)

After implementation, open a Codex session in this repo directory and verify:

- [ ] Type `$prp-commit` — Codex recognizes skill and executes
- [ ] Type `$prp-plan implement a new feature` — plan is created
- [ ] Type `$codebase-explorer` — agent skill executes
- [ ] `/skills` browser shows all 23 skills

### Edge Cases

- [ ] `prp-implement` correctly reads plan file path from user's natural language request
- [ ] Agent skills (`$codebase-explorer`) invoked by `prp-plan` execute as expected
- [ ] `AGENTS.md` loaded in Codex on session start (check Codex context indicator)

---

## Validation Commands

### Level 1: File Presence

```bash
# All 23 skill directories exist
ls .agents/skills/ | wc -l  # expect 23

# AGENTS.md exists
test -f AGENTS.md && echo "OK"

# CODEX.md exists
test -f CODEX.md && echo "OK"
```

### Level 2: Frontmatter Validity

```bash
# Every SKILL.md has name: field
for f in .agents/skills/*/SKILL.md; do
  grep -q '^name:' "$f" || echo "FAIL: missing name in $f"
done

# Every SKILL.md has description: field
for f in .agents/skills/*/SKILL.md; do
  grep -q '^description:' "$f" || echo "FAIL: missing description in $f"
done

# No agent-specific fields leaked into skill files
grep -r '^model:' .agents/skills/ && echo "FAIL: model: found" || echo "PASS"
grep -r '^color:' .agents/skills/ && echo "FAIL: color: found" || echo "PASS"
```

### Level 3: Content Cleanliness

```bash
grep -r '\$ARGUMENTS' .agents/skills/ && echo "FAIL" || echo "PASS: no $ARGUMENTS"
grep -r 'subagent_type' .agents/skills/ && echo "FAIL" || echo "PASS: no subagent_type"
grep -r 'SlashCommand tool' .agents/skills/ && echo "FAIL" || echo "PASS: no SlashCommand"
```

### Level 4: Manual Codex Session

Open Codex in this repo, confirm skills appear in `/skills` browser.

---

## Acceptance Criteria

- [ ] 23 SKILL.md files created (12 command skills + 11 agent skills)
- [ ] `AGENTS.md` created with Codex-appropriate language
- [ ] `CODEX.md` install README created
- [ ] Zero `$ARGUMENTS` tokens in any `.agents/skills/` file
- [ ] Zero `subagent_type` references in any `.agents/skills/` file
- [ ] Zero `SlashCommand tool` references in any `.agents/skills/` file
- [ ] All Claude Code files in `.claude/` remain unchanged
- [ ] Level 1-3 validation passes

---

## Completion Checklist

- [ ] Task 1: Directory scaffolding (23 dirs)
- [ ] Task 2: AGENTS.md created
- [ ] Tasks 3-14: 12 command skills created and clean
- [ ] Tasks 15-25: 11 agent skills created and clean
- [ ] Task 26: All validation passes
- [ ] Task 27: CODEX.md install guide created

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| prp-implement bash $ARGUMENTS replacements missed | M | M | Explicit 7-occurrence count in task; validate with grep |
| Agent sub-agent invocation syntax not recognized by Codex | M | M | Documented in open questions; natural language `$skill-name` is the accepted pattern |
| `.agents/skills/` not auto-discovered without config | L | H | Codex docs confirm auto-discovery; fallback: add path to `~/.codex/config.toml` |
| AGENTS.md Codex-specific content gaps | L | L | Users can extend manually; documented as DRAFT |

---

## Notes

**Why `$skill-name` replaces `Task tool with subagent_type`**: The Codex pattern for cross-skill invocation is natural language - "Use the `$skill-name` skill with the following prompt." This is the pattern used in the official `gh-fix-ci` skill from `github.com/openai/skills`. Codex resolves `$skill-name` references to loaded skills at runtime.

**AGENTS.md vs CLAUDE.md**: These are functionally identical formats. The only real differences are: (1) title/opening sentence, (2) invocation prefix (`$` vs `/`), (3) removal of unsupported skills (ralph).

**Phase 4 (live Codex validation) is manual**: After this plan is executed, the implementer or developer needs to open a real Codex session and run the skill smoke tests. If sub-agent invocation doesn't work as expected, the skill bodies may need minor adjustments to the invocation phrasing.

**prp-core-runner update**: The existing `.claude/skills/prp-core-runner/SKILL.md` references `SlashCommand tool` with `/prp-core-run-all`. The new `.agents/skills/prp-core-runner/SKILL.md` replaces this with sequential skill invocations. The `.claude/` version is NOT modified.
