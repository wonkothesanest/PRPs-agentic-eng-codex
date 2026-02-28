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
