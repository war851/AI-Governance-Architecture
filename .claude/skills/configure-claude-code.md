---
name: configure-claude-code
description: Use when the user wants to configure Claude Code itself — add new steps/functions/messages, set up skills/agents/hooks, configure GitHub/CI/TDD, or bootstrap a new project.
---

# Configure Claude Code

Meta-skill for configuring Claude Code's own environment: skills, agents, hooks, rules, slash commands, and project setup.

## How This Skill Works

When the user expresses a configuration goal, follow this pattern:

1. **Identify the intent** from the table below
2. **Enter Plan mode** — plan the phased approach before executing
3. **Reference the correct docs** — architecture skill, configuration guide, configuration sources
4. **Produce or update** the appropriate files (skills, rules, docs)

Always respect the Cursor/Claude Code separation:
- **Cursor** owns code standards, naming, architecture patterns (`.cursor/rules/`)
- **Claude Code** owns planning, execution, skills, agents, hooks, commands (`.claude/`)
- **Skills** live in `.claude/skills/` (Cursor auto-imports these)
- **Shared layer** owns plans, tasks, logs, knowledge base (`docs/`)

## Intent Routing Table

### Governed Application — Table-Driven Pattern

| User Says | Action Path |
|-----------|-------------|
| "Add new steps" / "new step" | 1. Plan: add row to `steps` table (migration). 2. If step has method_ref: add `methods` row. 3. If step has message_ref: add `static_messages` rows. 4. Add test coverage. |
| "Add new functions" / "new method" | 1. Plan: add row to `methods` table (migration). 2. If method needs a serverless function: create it. 3. Define I/O schema and link to step. |
| "Add new messages" / "new template" | 1. Plan: add rows to `static_messages` table (migration, with language variants). 2. Verify message_ref links from steps are correct. |

### Skills, Agents, and Automation

| User Says | Action Path |
|-----------|-------------|
| "Add a new skill" | 1. All skills go in `.claude/skills/` (Cursor auto-imports these). 2. Create with YAML frontmatter (name, description). 3. If it produces artifacts: add templates to `artifact-template.md`, register in `DESIGN.md`. |
| "Add an agent" / "subagent" | 1. Create `.claude/agents/<name>.md` with frontmatter (name, description, tools, permissionMode, maxTurns, memory). 2. Decide: `memory: "project"` for persistent context, `memory: "none"` for ephemeral. |
| "Add hooks" / "automation" | 1. Add hook config to `.claude/settings.json` (team) or `.claude/settings.local.json` (personal). 2. Hook types: PreToolUse (validate/block), PostToolUse (format/lint/test), Stop (notify). 3. Strategy: block at commit time, not at write time (per Boris best practice). 4. Reference: hooks guide at https://code.claude.com/docs/en/hooks-guide. |
| "Add a slash command" | 1. Create `.claude/commands/<name>.md` with YAML frontmatter (description). 2. Content: step-by-step workflow instructions. 3. Use for workflows repeated many times per day. |

### Testing and Quality

| User Says | Action Path |
|-----------|-------------|
| "Set up TDD" | 1. Rule: every phase that changes behavior must have a passing test. 2. Tests first: write test, see it fail, implement, see it pass. 3. For LLM-involving functions: use golden examples with evaluation criteria (see TDD Golden Examples skill). |
| "Security" / "access control" | 1. Every table with user data MUST have row-level security. 2. Migration includes: DDL + access policies + triggers + seed data. 3. Reference architecture skill: "Security by Default" principle. |

### External Services

| User Says | Action Path |
|-----------|-------------|
| "GitHub" / "Actions" / "CI" | 1. GitHub CLI: `gh` for PRs, issues, checks, releases. 2. Protected branches: main (prod). 3. Settings deny: `git push` blocked by default — user must explicitly approve. |

### Project Bootstrap

| User Says | Action Path |
|-----------|-------------|
| "New project" / "bootstrap" | Follow the 8-step checklist from the configuration guide: CLAUDE.md → directories → team settings → Cursor rules → skills → architecture skill → DESIGN.md → iterate. |
| "Configure from scratch" | Same as bootstrap. Reference the configuration guide. |

## Reference Documents

This skill draws from these sources (load on demand, never inline):

| Document | When to Reference |
|----------|-------------------|
| governed-ai-architecture.md (skill) | Any design decision, table changes, new steps/methods |
| claude-code-configuration-guide.md | 5-level cascade details, settings.json, hooks, agents, bootstrap checklist |
| claude-code-configuration-sources.md | Canonical URLs for Anthropic docs, Boris patterns |
| DESIGN.md | Application design status, artifact registry, skill connections |
| artifact-template.md (skill) | Creating new artifacts — YAML frontmatter envelope, per-type content templates |

## 5-Level Cascade (Quick Reference)

From the configuration guide — use this when deciding where config belongs:

| Level | File | Scope | Loaded |
|-------|------|-------|--------|
| 1 | `/etc/claude-code/CLAUDE.md` | Org-wide (IT-deployed) | Always, non-overridable |
| 2 | `~/.claude/CLAUDE.md` | Personal, all projects | Always |
| 3 | `./CLAUDE.md` | Team-shared, this project | Always |
| 4 | `.claude/rules/*.md` | Modular, path-scoped | Always (path-scoped on match) |
| 5 | `./CLAUDE.local.md` | Personal, this project | Always, gitignored |

Plus: settings.json (permissions, hooks), skills (on demand), agents (on spawn), commands (on invoke), auto memory (first 200 lines).

## Validation Before Any Change

- [ ] Maintains table-driven pattern (no hardcoded step logic)?
- [ ] One LLM call per user message?
- [ ] Environment variables for all secrets/URLs?
- [ ] Access control / row-level security considered?
- [ ] Migration created for schema changes?
