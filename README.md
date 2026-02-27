# Governed AI Architecture: Structure Beats Intelligence

> A reusable pattern where a single database owns the process — steps, functions, skills, messages, and data evolution — and the LLM is a stateless semantic engine. Don't train your AI. Certify it.

**[Open Interactive Architecture Diagram](docs/visualizations/Governed%20AI%20Architecture.html)**

---

## The Bridge: Claude Code → DB-Governed Applications

This architecture is a direct transformation of [Claude Code's configuration model](https://docs.anthropic.com/en/docs/claude-code/memory) into a database-governed application pattern. The same separation of concerns that makes Claude Code effective — structure over intelligence, config over code, governance over vibes — applied to full applications.

| Claude Code | DB Governed App | Same Principle |
|---|---|---|
| **CLAUDE.md cascade** — 5-level config hierarchy | **Config tables** — steps, messages, skills | Behaviour lives in config, not in code or weights |
| **Skills files** — methodology loaded at runtime | **Skills table** — methodology loaded at runtime | Don't train — certify. Swap a skill, change the lens |
| **Slash commands** — executable units by name | **Edge functions** — executable units by step | Defined input → defined logic → defined output |
| **Rules** — path-scoped, loaded when relevant | **Step scope** — prerequisites + visibility | Right context at the right moment |
| **PLAN.md** — explicit sequence of work | **Steps table** — explicit ordered sequence | Process is defined, not improvised by the LLM |
| **TODO.md** — tracks what feeds what | **Data evolution** — output of step N = input of step N+1 | The value chain of information is explicit |
| **Auto memory** — persisted across sessions | **Chat transcript + audit log** — persisted, queryable | Nothing lost in the context window |
| **Agents** — independent sub-workers | **Sub-agents** — parallel skill execution | Each owns its workspace, orchestrator coordinates |
| **settings.json** — permissions, allowed tools | **Function registry** — type, I/O schema, linked skill | What's allowed, what type, what contract |
| **Git history** — full version trail | **Audit log + versioned datasets** — full trace | Every decision reconstructable |
| **Hooks** — pre/post gates around actions | **Orchestrator loop** — before/after gates per step | Automated quality control around every execution |

---

## The 6 Principles

These are the constitution — like the managed policy layer in Claude Code, they cannot be overridden.

1. **DB owns the process, LLM is the semantic engine.** The database defines every step, message, function, and skill. The LLM receives a constrained prompt and returns structured output. It never decides what happens next.
2. **Don't train — certify.** Skills are runtime config. Swap a skill row, change the methodology. No retraining, no redeployment.
3. **Data evolution is the value chain.** The output of step N is the input of step N+1. Map this chain before writing any code.
4. **Every LLM call is constrained and testable.** Expected input + detailed prompt + skill config = structured output. Every call gets TDD tested with golden examples.
5. **Everything is stored.** Input as-is, output structured, full audit trail. Nothing lives only in the context window.
6. **LLM-agnostic.** Swap the provider, nothing else changes. The governance model is in the DB, not in the model.

---

## Why Governance Matters

When you govern your AI application at the database level, you get these as side effects — not extra work:

- **Auditability** — every prompt, every response, every decision is reconstructable from DB state at any point in time
- **EU AI Act / Model Risk** — you can show process definition, data lineage, and test evidence for every LLM call
- **GDPR** — field-level data evolution tells you exactly what personal data you have, where it lives, and how to delete it
- **LLM-agnostic** — swap providers without touching application logic; the governance model is in the DB, not the model

---

## The Design Skills

Four skills take you from first stakeholder interview to running application. Each produces governed artifacts that feed the next.

| Skill | Type | What It Does | Artifacts |
|---|---|---|---|
| [Value Chain Mapping](.claude/skills/value-chain-mapping.md) | Discovery | Maps the process from first input to final output through structured interviews | 4 types (value-chain-map, step-spec, stakeholder-register, gap-log) |
| [Data Evolution](.claude/skills/data-evolution.md) | Design | Traces field-level data transformations; lets the schema emerge from the flow | 5 types (evolution-diagram, state-diagram, evolution-table, transformation-audit, schema-derivation) |
| [TDD Golden Examples](.claude/skills/tdd-golden-examples.md) | Testing | Defines testable contracts for every function, including LLM calls | 3 types (golden-example-set, test-spec, eval-rubric) |
| [Governed Architecture](.claude/skills/governed-ai-architecture.md) | Architecture | Designs the DB-governed application from confirmed upstream artifacts | 5 types (steps-table, edge-functions, skills-config, static-messages, orchestrator-design) |

All 17 artifact types share a standard [YAML envelope](.claude/skills/artifact-template.md) and lifecycle: `draft` → `detailed` → `confirmed`.

---

## See It in Action

The [JD Parsing Example](docs/explanations/jd-parsing-example.md) walks through a real implementation from a hiring engine: a Job Description PDF is uploaded, parsed into structured items by an edge function (code+LLM), stored as DB rows, and later used to question the candidate. It demonstrates steps, invisible auto-execution, the code/LLM division of labor, and data evolution across tables.

---

## What's in This Workspace

**Root governance files (3):**
- `CLAUDE.md` — Team-shared project memory (dual-agent architecture, coding standards, Boris patterns, compounding engineering)
- `CLAUDE.local.md` — Personal local overrides (auto-gitignored)
- `DESIGN.md` — Application design governance index (artifact registry, skill connections, lifecycle tracking)

**Claude Code configuration (`.claude/`):**
- `settings.json` — Permissions (safe git commands allowed, secrets/destructive ops denied), PreToolUse hook blocking dangerous commands, Stop hook with notification, telemetry disabled
- 2 agents: `code-reviewer.md` (read-only reviewer), `planner.md` (read-only technical architect)
- 3 commands: `/plan`, `/review`, `/learn` (compounding engineering loop)
- 4 rules: `code-quality.md`, `git-workflow.md`, `planning-discipline.md`, `windows-environment.md`
- 6 skills: `configure-claude-code.md` (meta-skill), `value-chain-mapping.md`, `data-evolution.md`, `tdd-golden-examples.md`, `governed-ai-architecture.md`, `artifact-template.md`

**Cursor configuration (`.cursor/rules/`):**
- 3 rules: `project-conventions.mdc`, `planning-and-execution.mdc`, `git-standards.mdc`

**Documentation (`docs/`):**
- `claude-code-configuration-guide.md` — Comprehensive [5-level cascade guide](docs/claude-code-configuration-guide.md) with templates, examples, anti-patterns, and Boris patterns
- `claude-code-configuration-sources.md` — [Reference URLs](docs/claude-code-configuration-sources.md) for Anthropic docs and Boris's site
- Interactive visualizations — skill ecosystem explorer and governed architecture bridge (React)

For a deep walkthrough of every file, see [Workspace Anatomy](docs/explanations/workspace-anatomy.md).

---

## Quick Start

1. **Clone** this repository into your project root
2. **Edit** `CLAUDE.md` — fill in your project-specific details (stack, commands, architecture)
3. **Edit** `CLAUDE.local.md` — add your local environment specifics
4. **Customize** `.claude/settings.json` permissions for your project's tooling
5. **Start coding** — both Cursor and Claude Code pick up the configuration automatically

---

## The Development Environment

The application architecture above is the point of this repo. The development environment underneath it follows [Boris's best practices for Claude Code](https://howborisusesclaudecode.com/) and [Anthropic's configuration cascade](https://docs.anthropic.com/en/docs/claude-code/memory):

- **Cursor** owns code standards, naming conventions, and architecture patterns
- **Claude Code** owns planning, execution, skills, agents, hooks, and commands
- **Shared layer** owns CLAUDE.md, DESIGN.md, plans, tasks, and documentation
- **Compounding engineering** — every correction updates the config so the mistake never recurs

---

## Origin

This pattern was built from a working hiring engine ([System 01 — The Hiring Engine](https://01.toldi.io/)) — a governed AI application with ~15 steps, structured outputs, and full audit trails. To make it enterprise-grade and auditable, the methodology was pulled out of the runtime into explicit design skills and a database-governed orchestrator pattern, then generalized into this project-agnostic workspace.

---

## Sources

- [Anthropic: Claude Code Memory & CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Anthropic: Agentic Coding Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Boris's Claude Code Patterns](https://howborisusesclaudecode.com/)

---

## Author

**Csaba Toldi** — [LinkedIn](https://www.linkedin.com/in/csabatoldi/) · [toldi.io](https://toldi.io)

---

## License Scope

This repository contains three layers of intellectual property, each with different status:

### Public Background — Claude Code Best Practices

The development environment configuration (`.claude/settings.json`, rules, commands, agents, hooks) and the configuration guides in `docs/` are based on publicly available material from [Anthropic](https://docs.anthropic.com/en/docs/claude-code/memory) and [Boris Cherny](https://howborisusesclaudecode.com/). These are attributed throughout and are not claimed as original work.

### The Invention — Governed AI Methodology (CC BY-NC 4.0)

The original contribution of this repository is applying the Claude Code configuration model to database-governed application design. This includes:

- **The pattern:** a single database owns the process — steps, functions, skills, messages, and audit logs as the single source of truth, with the LLM as a stateless semantic engine
- **The four design skills:** Value Chain Mapping, Data Evolution, TDD Golden Examples, and Governed AI Architecture
- **The artifact system:** templates, YAML envelope standard, DESIGN.md governance, and the artifact-to-DB integration pattern
- **The 6 constitutional principles** and the bridge table mapping Claude Code concepts to DB-governed equivalents

This methodology is licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). You are free to study, share, and adapt it for non-commercial purposes with attribution. Commercial use of the methodology — embedding it in a product, selling consulting or training based on it — requires a commercial license.

### Application Code — Private

[System 01 — The Hiring Engine](https://01.toldi.io/) is a separate commercial product built on this methodology. The [JD Parsing Example](docs/explanations/jd-parsing-example.md) in this repo is one small proof slice; the full application code is not published here.

### Commercial Licensing

For commercial use of the Governed AI methodology, [get in touch](https://www.linkedin.com/in/csabatoldi/) or email [c@toldi.io](mailto:c@toldi.io).
