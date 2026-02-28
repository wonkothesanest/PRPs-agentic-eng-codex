# PRP Framework - OpenAI Codex Port

## Problem Statement

Developers who use OpenAI Codex cannot access the PRP framework's structured workflow commands because they are built exclusively for Claude Code's `.claude/commands/` system. Porting the framework makes the same methodology available to Codex users without requiring them to switch editors.

## Evidence

- This repo already contains one cross-compatible skill (`.claude/skills/prp-core-runner/SKILL.md`), proving the SKILL.md format is viable here
- SKILL.md is an open standard (Anthropic-originated, adopted by OpenAI Dec 2025) - skills are cross-compatible between Claude Code and Codex with no modification to the format
- AGENTS.md and CLAUDE.md are functionally identical - a direct copy-and-adapt is sufficient

## Proposed Solution

Convert the 12 active PRP commands into Codex-compatible SKILL.md files under `.agents/skills/`, create an `AGENTS.md` from `CLAUDE.md`, port the 11 sub-agent definitions to Codex agent format, and provide an installation README. No new abstractions - just format translation.

## Key Hypothesis

We believe converting commands to SKILL.md format will give Codex developers access to ~80% of PRP workflow commands. We'll know we're right when a developer can clone this repo, follow the install steps, and successfully run `$prp-plan`, `$prp-implement`, and `$prp-commit` inside a Codex session.

## What We're NOT Building

- **Ralph mode** - depends on lifecycle hooks (PreToolUse/PostToolUse) that Codex does not support in production
- **Marketplace submission** - out of scope; developers install from the repo directly
- **Claude Code backward-compatibility shims** - existing `.claude/` structure stays untouched
- **Custom Codex config.toml** - no project-specific config needed for this port

## Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
| Commands available in Codex | 11 of 13 (85%) | Manual invocation test |
| Sub-agents functioning | All 11 best-effort | Run each skill end-to-end once |
| Install steps | ≤ 3 commands | Count steps in README |

## Open Questions

- [ ] Codex sub-agent spawning syntax - needs validation against live Codex session
- [ ] Whether `.agents/skills/` at repo root is auto-discovered by Codex CLI or needs config
- [ ] If AGENTS.md needs any Codex-specific additions beyond CLAUDE.md content

---

## Users & Context

**Primary User**
- **Who**: Software developer who uses OpenAI Codex as their primary AI coding tool
- **Current behavior**: Manually copies PRP templates or skips the framework entirely
- **Trigger**: Wants structured PRP workflow (plan → implement → commit → PR) inside Codex
- **Success state**: Types `$prp-plan` and gets the same result they'd get from `/prp-plan` in Claude Code

**Job to Be Done**
When I'm in a Codex session and want to run a PRP workflow, I want to invoke skills that follow the PRP methodology, so I can ship features using the same structured process without switching tools.

**Non-Users**
- Claude Code users - existing `.claude/` structure is not being changed
- Teams needing enterprise org deployment - out of scope

---

## Solution Detail

### Core Capabilities (MoSCoW)

| Priority | Capability | Rationale |
|----------|------------|-----------|
| Must | Convert 12 commands → SKILL.md under `.agents/skills/` | Core deliverable |
| Must | Create `AGENTS.md` from `CLAUDE.md` | Project context injection |
| Must | Port 11 sub-agent definitions to Codex agent format | Commands depend on agents |
| Must | Installation README | Required for usability |
| Should | Validate skills work end-to-end in Codex | Ensures 80% target |
| Won't | Ralph / hooks-dependent commands | Codex hooks not production-ready |
| Won't | Marketplace submission | Out of scope |

### MVP Scope

All 12 non-Ralph commands converted and loadable in Codex. Developer can clone repo and follow a README to activate skills.

### Commands to Port (12 of 13)

| Command | Skill Name | Notes |
|---------|------------|-------|
| `prp-plan` | `$prp-plan` | Direct port |
| `prp-implement` | `$prp-implement` | Direct port |
| `prp-commit` | `$prp-commit` | Direct port |
| `prp-pr` | `$prp-pr` | Direct port |
| `prp-review` | `$prp-review` | Direct port |
| `prp-review-agents` | `$prp-review-agents` | Direct port |
| `prp-debug` | `$prp-debug` | Direct port |
| `prp-prd` | `$prp-prd` | Direct port |
| `prp-codebase-question` | `$prp-codebase-question` | Direct port |
| `prp-issue-investigate` | `$prp-issue-investigate` | Direct port |
| `prp-issue-fix` | `$prp-issue-fix` | Direct port |
| `prp-core-runner` | `$prp-core-runner` | Already exists as SKILL.md - update |
| ~~`prp-ralph`~~ | - | Skipped - hooks dependency |
| ~~`prp-ralph-cancel`~~ | - | Skipped - hooks dependency |

### Sub-Agents to Port (11)

`.claude/agents/` → `.agents/skills/agents/` or inline in skill definitions:

- web-researcher, codebase-explorer, codebase-analyst, code-reviewer
- code-simplifier, comment-analyzer, docs-impact-agent, pr-test-analyzer
- silent-failure-hunter, type-design-analyzer, gpui-researcher

### Skill File Format

Each `.agents/skills/{skill-name}/SKILL.md`:

```yaml
---
name: prp-plan
description: Create a comprehensive feature implementation plan with codebase analysis and research. Use when a developer wants to plan implementation of a feature or task.
---

{existing command content verbatim}
```

### User Flow

```
git clone <repo> → follow README install step → open Codex in project → type $prp-plan
```

---

## Technical Approach

**Feasibility**: HIGH

The SKILL.md format requires only adding YAML frontmatter (name + description) to existing Markdown command files. The content body is identical. No logic changes needed.

**Architecture Notes**
- `.agents/skills/` directory at repo root for Codex auto-discovery
- Each skill is a subdirectory with a single `SKILL.md`
- `AGENTS.md` = `CLAUDE.md` with minor Codex-specific additions (if any discovered during validation)
- Sub-agents: Codex agent spawning uses different syntax - best-effort adaptation with validation

**Technical Risks**

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Codex sub-agent syntax differs significantly from Claude Code | M | Test one agent early; adjust pattern for rest |
| `.agents/skills/` not auto-discovered without config | L | Add path to `config.toml` if needed |
| Some commands reference Claude Code-specific tools (Task, Bash, etc.) | M | Replace with Codex equivalents where needed; document gaps |

---

## Implementation Phases

| # | Phase | Description | Status | Parallel | Depends | PRP Plan |
|---|-------|-------------|--------|----------|---------|----------|
| 1 | Scaffold | Create `.agents/skills/` directory structure and `AGENTS.md` | complete | - | - | `.claude/PRPs/plans/completed/prp-framework-codex-port-phase1.plan.md` |
| 2 | Skills Conversion | Add YAML frontmatter to all 12 commands, place in `.agents/skills/` | complete | with 3 | 1 | `.claude/PRPs/plans/completed/prp-framework-codex-port-phase1.plan.md` |
| 3 | Agent Port | Convert 11 sub-agent definitions to Codex-compatible format | complete | with 2 | 1 | `.claude/PRPs/plans/completed/prp-framework-codex-port-phase1.plan.md` |
| 4 | Validation | Test each skill in a live Codex session, document gaps | pending | - | 2, 3 | - |
| 5 | Install README | Write installation and usage guide | complete | - | 4 | `.claude/PRPs/plans/completed/prp-framework-codex-port-phase1.plan.md` |

### Phase Details

**Phase 1: Scaffold**
- **Goal**: Create the directory structure Codex expects
- **Scope**: `mkdir .agents/skills/`, create `AGENTS.md` from `CLAUDE.md`
- **Success signal**: `AGENTS.md` exists, `.agents/skills/` directory exists

**Phase 2: Skills Conversion**
- **Goal**: 12 skills loadable in Codex
- **Scope**: For each command file, create `.agents/skills/{name}/SKILL.md` with YAML frontmatter + existing body
- **Success signal**: All 12 skills appear in Codex `/skills` browser

**Phase 3: Agent Port**
- **Goal**: Sub-agents callable from within skills
- **Scope**: Convert `.claude/agents/*.md` to Codex agent format in `.agents/skills/agents/`
- **Success signal**: At least 8 of 11 agents invoke correctly in a test skill run

**Phase 4: Validation**
- **Goal**: 80% command success rate confirmed
- **Scope**: Run each skill once end-to-end on a test repo, log what works/fails
- **Success signal**: 11+ of 13 skills produce correct output

**Phase 5: Install README**
- **Goal**: A developer with no prior context can get running in < 5 min
- **Scope**: Installation steps, usage guide, known gaps documented
- **Success signal**: Fresh install works following README only

### Parallelism Notes

Phases 2 and 3 can run in parallel (separate worktrees if desired) since skills and agents touch different directories and have no interdependency during conversion.

---

## Decisions Log

| Decision | Choice | Alternatives | Rationale |
|----------|--------|--------------|-----------|
| Ralph exclusion | Skip | Stub with warning | Hooks not available; stub adds confusion |
| Install method | Clone repo | Publish to SkillsMP | Faster; marketplace submission is separate effort |
| Agent format | Best-effort port | Skip agents | Agents required for most skills to function |
| AGENTS.md source | Adapt from CLAUDE.md | Write from scratch | CLAUDE.md already has correct content |

---

## Research Summary

**Market Context**
- SKILL.md is a shared open standard between Claude Code and Codex (Anthropic-originated, Dec 2025)
- SkillsMP hosts 270k+ community skills using the same format
- Codex auto-discovers skills in `.agents/skills/` without any config in most cases

**Technical Context**
- Repo has 13 commands in `.claude/commands/prp-core/`, 11 agents in `.claude/agents/`, 1 existing skill in `.claude/skills/`
- Hooks (`.claude/hooks/`) are Ralph-specific - skipping entirely
- Conversion is additive - existing Claude Code files remain unchanged

---

*Generated: 2026-02-28*
*Status: DRAFT - needs validation*
